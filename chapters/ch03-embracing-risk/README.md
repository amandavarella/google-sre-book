# Chapter 03 — Embracing Risk

> Part II. Principles of *Site Reliability Engineering* (Google / O'Reilly).
>
> Notes last synced 2026-08-14.

---

<!-- obsidian:start -->

## Notes

### Managing Risk

Rather than simply maximizing uptime, SRE seeks to balance the risk of unavailability against the
goals of rapid innovation and efficient service operations, so that users' overall happiness (with
features, service, and performance) is optimized.

- Make a service reliable enough, but no *more* reliable than it needs to be. When you set an
  availability target (e.g., 99.99%), you want to exceed it, but not by much: overshooting wastes
  opportunities to add features, clean up technical debt, or reduce operational costs.
- Treat the availability target as both a minimum and a maximum. The key advantage of this framing
  is that it unlocks explicit, thoughtful risk-taking.

### Measuring Service Risk

Standard practice at Google is to identify an objective metric for the property you want to
optimize, so you can assess current performance and track improvements or degradations over time.
Service risk is hard to reduce to a single metric (failures cause user dissatisfaction, harm, loss
of trust, revenue loss, brand/reputational impact, bad press), so to make it tractable and
consistent, SRE focuses on **unplanned downtime**.

- Unplanned downtime is captured by the desired level of *service availability*, usually expressed
  in "nines" (99.9%, 99.99%, 99.999%).
- **Time-based availability.** Over a year this yields an acceptable downtime budget (e.g., 99.99%
  ≈ 52.56 minutes/year; see Appendix A):

*Equation 3-1. Time-based availability*

$$\text{availability} = \frac{\text{uptime}}{\text{uptime} + \text{downtime}}$$

- At Google a purely time-based metric is usually not meaningful, because globally distributed
  services are almost always at least partially "up". Instead they use a **request success rate**,
  or aggregate availability, computed over a rolling window:

*Equation 3-2. Aggregate availability*

$$\text{availability} = \frac{\text{successful requests}}{\text{total requests}}$$

- Example: a service serving 2.5M requests/day with a 99.99% target can serve up to 250 errors
  that day and still hit the target.
- The request-success-rate framing also applies to non-serving systems (batch, pipeline, storage)
  that have a well-defined notion of successful vs unsuccessful units of work.

### Risk Tolerance of Services

Identifying a service's risk tolerance means turning business goals into explicit objectives you
can engineer to. In safety-critical or formal environments the risk tolerance is often built into
the product definition; at Google it tends to be less clearly defined, so SREs work with the
product owners to make it explicit.

- Consumer services usually have clear product owners (Search, Maps, Docs each have product
  managers), so there's a natural partner to define reliability requirements.
- Infrastructure services (storage systems, a general-purpose HTTP caching layer) rarely have that
  kind of product ownership, so the risk tolerance has to be derived differently.

#### Consumer services

##### Types of failures

- Failure modes differ in severity: a poor user experience is remediated quickly, but exposing
  private data (e.g., showing one user's data to another) could undermine basic user trust, so
  taking the service down during debugging and cleanup can be the right call.
- At the other extreme, some regular outages are acceptable and count as *planned* downtime rather
  than unplanned. Years ago the Ads Frontend was one such service: advertisers and website
  publishers use it to set up, configure, run, and monitor their advertising campaigns, and because
  almost all of that work happens during normal business hours, Google decided that occasional,
  regular, scheduled maintenance windows were acceptable. Those scheduled outages were classified
  as planned downtime, so they did not count against the service's unplanned-downtime
  (availability) budget.

##### Cost

Cost is often the key factor in setting the availability target. Ads can translate request
successes/failures directly into revenue, so they ask: at one more nine of availability, what's the
incremental revenue, and does it offset the cost of reaching that reliability?

- Worked example: improving 99.9% → 99.99% adds 0.09% availability; on $1M service revenue that's
  `$1M × 0.0009 = $900`. If the extra nine costs less than $900 it's worth it; more than $900 and
  it isn't.
- Setting the target is harder when there's no simple translation function between reliability and
  revenue. One useful strategy is to compare against the **background error rate of ISPs** on the
  internet. If failures are measured from the end-user's perspective and you can drive the
  service's error rate below that background rate, those service errors fall within the noise of
  the user's own internet connection, so making the service even more reliable buys nothing the
  user can perceive. Google has measured the typical ISP background error rate at roughly **0.01%
  to 1%** (it varies significantly by ISP and protocol, e.g., TCP vs UDP, IPv4 vs IPv6), which
  gives a practical floor for how much service reliability is actually worth pursuing.

##### Other service metrics

Examining risk tolerance against metrics beyond availability is often fruitful:

- Web Search's distinguishing feature was speed, so AdWords (ads next to search results) must not
  slow the search experience: this is treated as an invariant across generations of the system.
- AdSense (contextual ads inserted via publisher JavaScript) has a different latency goal: don't
  slow the third-party page render, so its target depends on the publisher's page speed. AdSense
  ads can be served hundreds of milliseconds slower than AdWords ads.
- That looser latency requirement lets Google consolidate serving into fewer geographic locations,
  saving substantial cost over naive provisioning.

#### Infrastructure services

Infrastructure components differ from consumer products: by definition they have multiple clients,
often with varying needs.

##### Target level of availability

- e.g., Bigtable. Some consumer services serve user-facing data directly from Bigtable (need low
  latency, high reliability); other teams use it as a repository for offline analysis such as
  MapReduce (care more about throughput than reliability). Their risk tolerances are distinct.
- Making all infrastructure ultra-reliable is usually far too expensive. To see the different
  needs, look at the desired request-queue state per user type.

##### Types of failures

- The low-latency user wants request queues almost always empty (each request processed
  immediately; inefficient queuing causes high tail latency). The throughput user wants queues
  never empty (the system never idles). Success for one is failure for the other.

##### Cost

- Partition the infrastructure and offer it at multiple independent service levels. In Bigtable,
  low-latency clusters are provisioned with slack capacity (reduced contention, increased
  redundancy), while throughput clusters run hot with less redundancy. The relaxed needs are met at
  roughly 10-50% of the cost of a low-latency cluster; at Bigtable's scale the savings are large.
- Key strategy: deliver services with explicitly delineated levels of service so clients make the
  right risk/cost trade-offs. Exposing cost this way motivates clients to choose the lowest-cost
  level that still meets their needs.
- Example: Google+ can put privacy-critical data in a high-availability, globally consistent
  datastore (a globally replicated SQL-like system like Spanner), while putting optional
  UX-enhancing data in a cheaper, less reliable, eventually consistent datastore (a NoSQL store
  with best-effort replication like Bigtable). You can run multiple classes of service on identical
  hardware and software, providing very different guarantees by adjusting resource quantities,
  redundancy, geographical provisioning constraints, and, critically, the infrastructure software
  configuration.

### Motivation for Error Budgets

To base risk decisions on objective data, the SRE and product-development teams jointly define a
**quarterly error budget** based on the service's SLO (see Chapter 4). The error budget is a clear,
objective metric for how unreliable the service is allowed to be in a quarter, which removes
politics from negotiations about how much risk to allow.

The practice:

- Product Management defines an SLO, setting an expectation of how much uptime the service should
  have per quarter.
- Actual uptime is measured by a neutral third party: the monitoring system.
- The difference between the two numbers is the "budget" of how much unreliability remains for the
  quarter.
- As long as measured uptime is above the SLO (there is error budget remaining), new releases can
  be pushed.

Example: if a service's SLO is to successfully serve 99.999% of queries per quarter, its error
budget is a 0.001% failure rate. A problem that fails 0.0002% of expected queries spends 20% of the
quarterly error budget.

## Key takeaways

- Managing service reliability is largely about managing risk, and managing risk can be costly.
  Reliability is a balancing act: aim for "reliable enough" (the target is both floor and ceiling),
  because extra nines cost real feature velocity and money.
- 100% is probably never the right reliability target: it's impossible to achieve, and typically
  more reliability than users want or notice. Match the profile of the service to the risk the
  business is willing to take.
- Measure risk with an objective metric. Prefer request success rate over time-based uptime for
  distributed services (fault isolation means such a service is almost always at least partially
  "up" somewhere in the world, so a time-based number says little about what users actually
  experienced).
- The same success-rate framing extends to systems that don't serve end users continuously, which
  is where uptime breaks down completely: a batch job, pipeline, storage, or transactional system
  isn't *supposed* to be running all the time, so "percentage of time up" is meaningless for it.
  What these systems do have is a well-defined notion of a successful vs unsuccessful **unit of
  work**, so you count those instead. Example: a periodic ETL job that extracts a customer database
  and loads it into a data warehouse can define availability as the proportion of records processed
  successfully, giving a useful availability number even though the job only runs occasionally.
- Risk tolerance is per-service and even per-client: consumer services weigh cost against revenue
  per nine, while infrastructure services serve conflicting low-latency vs high-throughput needs,
  best solved by offering explicit service tiers at different costs (e.g., Bigtable runs two kinds
  of clusters: low-latency ones provisioned with slack capacity and extra redundancy for user-facing
  reads, and throughput ones run hot with less redundancy for offline analysis, at roughly 10-50% of
  the cost. Publishing that price difference pushes each client to pick the cheapest tier that still
  meets its needs, instead of everyone demanding the gold tier "just in case").
- An error budget (1 − SLO) turns "how much risk?" into an objective, shared number. It aligns
  incentives and emphasizes joint ownership between SRE and product development, makes it easier to
  decide the rate of releases, defuses discussions about outages with stakeholders (e.g., instead of
  arguing over whether last week's incident was "unacceptable", you can say it spent 20% of this
  quarter's error budget and 80% remains, so releases continue: the conversation becomes arithmetic
  against an agreed number rather than blame), and lets multiple teams reach the same conclusion
  about production risk without rancor.

<!-- obsidian:end -->

## Interactive diagrams

_None yet._

## Questions / things to revisit

-
