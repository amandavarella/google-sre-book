# Chapter 04 — Service Level Objectives

> Part II. Principles of *Site Reliability Engineering* (Google / O'Reilly).
>
> Notes last synced 2026-08-28.

---

<!-- obsidian:start -->

## Notes

### Indicators

An **SLI** (service level indicator) is a carefully defined, quantitative measure of some aspect
of the level of service provided.

- The most common SLIs: **request latency** (time to return a response), **error rate** (usually
  a fraction of all requests), and **throughput** (typically requests/second). Raw measurements
  are usually aggregated over a window into a rate, average, or percentile.
- Ideally the SLI measures the service level you actually care about directly, but sometimes only
  a proxy is available because the real thing is hard to obtain or interpret. Example: client-side
  latency is usually the more user-relevant metric, but it may only be possible to measure latency
  at the server.
- **Availability** is another key SLI for SREs: the fraction of time a service is usable, often
  defined as the fraction of well-formed requests that succeed (sometimes called **yield**).
  **Durability** (the likelihood data will be retained over a long period) is the equivalent
  concept for storage systems.
- 100% availability is impossible, but near-100% is often achievable. The industry expresses this
  in "nines": 99% is "2 nines", 99.999% is "5 nines". Google Compute Engine's published target is
  "three and a half nines", 99.95%.

### Objectives

An **SLO** (service level objective) is a target value or range for a service level, measured by
an SLI. The natural structure is `SLI ≤ target` or `lower bound ≤ SLI ≤ upper bound` (e.g.,
"average search request latency should be under 100 milliseconds").

- Choosing an SLO is complex, and you don't always get to pick the value. Incoming traffic volume
  (QPS) is driven by what users want, so you can't really set an SLO for it. But you can set one
  for average latency per request, which in turn motivates concrete engineering choices
  (low-latency frontend code, faster hardware). Lower latency is generally good, since above
  certain thresholds user-experienced latency measurably drives users away (see "Speed Matters"
  [Bru09]).
- SLIs can be connected behind the scenes: higher QPS often drives up latency, and many services
  have a performance cliff beyond some load threshold. Setting one SLO can constrain what's
  achievable on the other.
- Choosing and **publishing** SLOs to users sets expectations for how the service performs, which
  reduces unfounded complaints (e.g., "it's slow"). Without an explicit SLO, users invent their
  own beliefs about expected performance, unrelated to what the people running the service
  actually believe.
  - This mismatch cuts both ways: **over-reliance**, when users wrongly assume a service is more
    available than it is (this happened with Chubby, see "The Global Chubby Planned Outage"
    below), and **under-reliance**, when prospective users assume a system is flakier than it
    actually is.

### The Global Chubby Planned Outage (case study)

Global Chubby is Google's distributed lock service, replicated so each instance spans multiple
geographical regions for resilience. It turned out to be so reliable that other teams started
building on it as if it could never go down, without ever building a fallback path for a Chubby
outage.

- **The trap**: high reliability created a false sense of security. Teams took on a hidden
  dependency (assuming Chubby is always available) that they never tested, because a real Chubby
  outage almost never happened.
- **Why that's dangerous**: the risk didn't go away, it just moved and compounded. When Chubby
  did eventually fail, every dependent service failed at once, in an uncontrolled, high-stakes
  moment, because none of them had ever exercised that failure path.
- **SRE's fix (counter-intuitive)**: deliberately keep global Chubby's actual reliability close
  to its SLO, not above it. If a quarter's real failures haven't already used up the error
  budget, SRE manufactures a planned outage on purpose to spend the rest of it.
- **Why breaking it on purpose helps**: it forces hidden dependencies to surface now, on a
  schedule, at low stakes, instead of later during a real incident. Teams find and fix fragile
  assumptions (missing retries, no caching, no graceful degradation) while things are calm.
- **The bigger idea**: an availability target is both a floor and a ceiling. Undershooting is an
  obvious problem, but overshooting isn't free either. Extra reliability handed out for free
  becomes a systemic risk that other people's systems quietly start depending on, one you no
  longer control. Deliberately breaking Chubby keeps that ceiling honest.
- **Industry parallel**: same philosophy as chaos engineering (e.g., Netflix's Chaos Monkey),
  exercise failure paths on your own schedule rather than discovering them for the first time
  during a real incident.

### Agreements

An **SLA** (service level agreement) is an explicit or implicit contract with users that includes
the consequences of meeting or missing the SLOs it contains.

- The consequences are most recognizable when financial (a rebate or a penalty), but can take
  other forms. A quick way to tell an SLO from an SLA: ask "what happens if the SLOs aren't met?"
  If there's no explicit consequence, it's an SLO, not an SLA.
- SRE doesn't typically build SLAs, since they're tied to business and product decisions. SRE
  does get involved in avoiding the consequences of missed SLOs, and in helping define the SLIs,
  since the agreement needs an objective way to measure whether the SLO was met, or disagreements
  follow.
- Example: Google Search has no public SLA (there's no contract with the whole world), but
  unavailability still has real consequences: reputation damage and lost advertising revenue.
  Other services, like Google for Work, do have explicit SLAs with their users.
- Whether or not a service has an SLA, defining SLIs and SLOs and using them to manage the
  service is valuable either way.

### Indicators in Practice

#### What Do You and Your Users Care About?

Don't turn every trackable metric into an SLI. Pick a handful of indicators that reflect what
users actually want from the system. Too many indicators makes it hard to pay attention to the
ones that matter; too few leaves real problems unexamined. A handful of representative indicators
is usually enough to reason about a system's health.

- **User-facing serving systems** (e.g., the Shakespeare search frontends): care about
  **availability**, **latency**, and **throughput**: could we respond to the request, how long
  did it take, how many requests could we handle?
- **Storage systems**: care about **latency**, **availability**, and **durability**: how long
  does it take to read/write data, can we access the data on demand, is the data still there when
  needed?
- **Big data systems** (e.g., data processing pipelines): care about **throughput** and
  **end-to-end latency**: how much data is being processed, how long does it take data to go from
  ingestion to completion (some pipelines also set latency targets on individual stages).
- **All systems** should care about **correctness**: was the right answer returned, the right
  data retrieved, the right analysis done? Correctness is worth tracking as a health indicator,
  even though it's usually a property of the data rather than the infrastructure, and so is often
  not an SRE responsibility to meet.

#### Collecting Indicators

Most indicator metrics are best gathered **server-side**, using a monitoring system such as
Borgmon or Prometheus, or via periodic log analysis, for example tracking **HTTP 500** responses
(the standard HTTP status code meaning the server hit an unexpected error while processing the
request, a server-side failure) as a fraction of all requests.

- Some systems also need **client-side collection**: instrumenting the actual client (e.g.,
  browser JavaScript, or a mobile app's SDK) to record what the user experienced, then reporting
  those measurements back to the service (e.g., via a lightweight logging/beacon call), rather
  than relying only on what the server can see.
- Why it matters: not measuring at the client can miss problems that hurt users but never show up
  in server-side metrics. Example: focusing only on the Shakespeare search backend's response
  latency could miss poor user-perceived latency caused by the page's JavaScript. Measuring how
  long it takes for the page to become usable in the browser is a better proxy for what the user
  actually experiences.

#### Aggregation

Aggregate raw measurements carefully. Most metrics are distributions, not averages.

- For a latency SLI, some requests finish quickly and others take much longer. A simple average
  hides those tail latencies, and hides changes in them.
- Figure 4-1: a typical request is served in about 50 ms, but 5% of requests are 20 times slower.
  If you monitor and alert only on average latency, the day looks unchanged, even while the tail
  (p95 / p99) is moving a lot. See [Latency percentiles](diagrams/latency-percentiles.html).

##### A Note on Statistical Fallacies

Percentiles are usually more useful than the arithmetic mean (the average). They show the long
tail, which often behaves differently from the middle of the data. Computer systems produce
skewed, bounded numbers: a request cannot finish in less than 0 ms, and a 1,000 ms timeout means
no successful response can be slower than that. So the mean and the median need not be the same,
or even close.

Do not assume the data follows a normal distribution (a bell curve) without checking. If the
shape is different, a rule that acts on outliers (restart a server with high request latencies)
will fire too often, or not often enough.

**Worked example: 100 requests, timeout at 1 second**

| Latency | Requests |
|---------|----------|
| 40 ms | 60 |
| 80 ms | 25 |
| 400 ms | 10 |
| timed out at 1 s | 5 |

- **Median = 40 ms.** Half the requests are at or below 40 ms, so the middle of the sorted list
  is still among the 60 requests that finished in 40 ms.
- **Mean = 134 ms.** `(60×40 + 25×80 + 10×400 + 5×1000) / 100 = 134`. The ten slow successes and
  five timeouts drag the average more than 3× past the median.

**What "the tail is chopped off" means**

A 1 second timeout is a rule the *service* enforces, not a fact about how long work takes. If a
request has not finished by 1,000 ms, the server (or the client, or the load balancer) gives up,
returns an error, and stops waiting.

That has two consequences for the latency histogram of *successful* requests:

1. **No success can be slower than 1,000 ms.** By definition a success is a request that finished
   in time. The histogram of successes therefore has a hard right wall at the timeout. The long
   tail you would have seen (1.2 s, 5 s, 30 s hangs) is not on the chart.
2. **You lose the real duration of the failures.** Those five timed-out requests might have
   finished at 1,001 ms if you had waited 1 more millisecond, or they might still be hung at 30 s.
   The metric does not distinguish.

**The problem with the two ways of recording those five**

You have to put them *somewhere*. Both choices distort the picture, in opposite directions.

- **Drop them from latency** (they are errors only). The latency chart is then *conditioned on
  success*: you threw out the worst users. Mean of the remaining 95 is 88 ms, not 134 ms, and p99
  looks like 400 ms. Latency dashboards and latency SLOs stay green. The 5% who got nothing only
  show up if someone is watching the error rate, and even then you do not see that those errors
  were "hung for a long time" vs "failed immediately." Slow-but-ok and failed-badly live on two
  different graphs, so a hang can look like a mild availability dip.
- **Record them as exactly 1,000 ms** (what this example does). A request that missed the deadline
  by 1 ms and a thread that would still be hung at 30 s occupy the same bar. Mean and p99 move a
  little, but nothing in the chart says "extreme." A restart-on-very-slow-request rule never sees
  a 30 s outlier, so it under-fires. You also mix failures into a success-latency number, which
  muddles "the service was slow" with "the service gave up."

Either way the true tail is missing, so you make the wrong call: you overreact to the visible
400 ms successes (they look like outliers on a bell curve) and underreact to the invisible hangs
(they never appear as 30 s). Computing systems *manufacture* this shape. If you treat the chart
as a natural distribution, your automation and your SLOs inherit the lie.

So "chopped off" is literal: someone took scissors to the right-hand side of the distribution at
1,000 ms. The 40 ms and 80 ms bars are real user experience. The empty space to the right of the
wall is not "we have no slow requests"; it is "we refused to keep measuring past 1 s."

The left side is chopped too: nothing can finish in less than 0 ms. Together, floor at 0 and
ceiling at the timeout are why this is not a bell curve, and why mean and median need not be
close.

See the six-step walkthrough: [Mean vs median](diagrams/statistical-fallacies.html).

**Why this breaks "restart on outlier" automation**

If you assume a bell curve without checking, a rule that "acts on outliers" (restart the slow
server) will fire at the wrong rate.

**What a bell curve and σ are**

A bell curve (normal distribution) is a shape where most values sit near the middle, and values
get rarer as you move away. σ (sigma) is one step of typical spread, the width of that bump.

The orange curve on step 5 is that assumed shape: centre 50 ms, σ = 20 ms. Read it as "we believe
a typical request is 50 ms, and 20 ms is a normal amount of jitter."

From the centre, count steps of 20 ms:

| How far from 50 ms | Latency | If it really were a bell curve |
|---|---|---|
| 1σ | 30–70 ms | about 68% of requests |
| 2σ | 10–90 ms | about 95% of requests |
| 3σ | 0–110 ms | about 99.7% of requests (the left side hits 0 ms) |

"3σ outlier" is shorthand for **slower than 50 + 3×20 = 110 ms**. Under a true bell curve, only
about **0.15%** of requests are that slow: roughly **1 in 700**.

**What our 100 requests actually do**

Slower than 110 ms in the table: the 10 at 400 ms plus the 5 timeouts = **15 of 100 = 15%**.

15% vs 0.15% is **100 times** more often than the model allows. A rule "restart the server if we
see latency over 110 ms" would treat ordinary tail traffic as a broken machine, constantly.

**Why they are false outliers**

An outlier, in the sense that restart-the-server automation cares about, means "this is so rare
it probably means the machine is sick." That is only true if your picture of "rare" is right.

The 10 requests at 400 ms **succeeded**. Users got an answer. A long tail of slow-but-ok requests
is normal for latency (locks, garbage collection, a cold cache, a bigger payload). Ten in a
hundred can be the everyday shape of the service, not a crisis. They look like 3σ events only
because the bell curve claims 110 ms is already "almost never." The name "outlier" is coming from
the model, not from a failure.

So they are **false** outliers: flagged as weird, actually ordinary tail. Acting on them
(restart, page, stop deploys) punishes a healthy system.

The 5 timeouts are different: those users did not get an answer. They are real failures. They
are still not "3σ of a bell curve." They are requests the timeout chopped at 1 s. Treating them
as statistical outliers of latency mixes up "slow success" and "gave up."

On the chart the orange bump dies out by 110 ms. The 400 ms and 1 s bars sit in the region the
bump says should be almost empty. That is the mismatch. The data is not a bell curve, so the
110 ms threshold is the wrong definition of "weird."

The timeout then breaks the same rule in the other direction: a server hung for 30 s is recorded
as 1,000 ms, same as a request that just grazed the deadline. The automation never sees a 30 s
outlier, so it does not restart when it should.

#### Standardize Indicators

Standardize on common SLI definitions so you don't have to reason about them from first
principles every time. Anything that matches the house template can be omitted from an
individual SLI.

The default knobs (the bits you stop repeating once they're standard):

- Aggregation interval: averaged over 1 minute
- Aggregation region: all the tasks in a cluster
- Measurement frequency: every 10 seconds
- Which requests: HTTP GETs from black-box monitoring jobs
- How data is acquired: through monitoring, measured at the server
- Data-access latency: time to last byte

Build a small set of reusable templates per common metric. Then a specific SLI is just the
service name plus any override.

**House defaults** (apply to every template below unless the SLI says otherwise): averaged over
1 minute, all tasks in the cluster, sampled every 10 seconds, measured at the server.

##### Template: Availability (serving)

Proportion of well-formed requests that succeed (not 5xx). Client errors (4xx) are excluded, they
are not the service failing.

- Full: "Proportion of well-formed HTTP GET requests to the Shakespeare search frontend that are
  not 5xx, measured at the server, averaged over 1 minute, across all tasks in the cluster."
- Once the template exists: **"Availability of Shakespeare search."**

##### Template: Latency (serving)

Time to last byte, as a distribution (p50 and p99), for HTTP GETs from black-box probing.

- Full: "99th percentile of time-to-last-byte for HTTP GET requests to Shakespeare search from
  black-box monitoring jobs, measured every 10 seconds, averaged over 1 minute."
- Short: **"Search frontend latency (p99)."**
- Payoff in an SLO: you can write `99% of Get RPC calls will complete in less than 100 ms`
  instead of `99% (averaged over 1 minute) of Get RPC calls will complete in less than 100 ms
  (measured across all the backend servers)`. Same meaning, because the parentheses are already
  in the template.

##### Template: Throughput (serving)

Successful requests per second, using the same request filter as availability.

- Short: **"Search QPS (successful)."**
- Note: incoming demand is not an SLO you set, but successful throughput is still a useful SLI
  for capacity and cliffs.

##### Template: Durability (storage)

Proportion of records written that can still be read later (is the data still there when we need
it).

- Full: "Proportion of objects successfully written to the photo store in the last 365 days that
  can be retrieved with a matching checksum, measured by a weekly audit job."
- Short: **"Photo store durability."**

##### Template: Pipeline freshness (batch / big data)

End-to-end time from ingestion to a completed, correct output record, plus records processed per
minute.

- Full: "99th percentile of time from a row landing in the ingest topic to it appearing in the
  serving table, excluding retries after a known poison-pill, averaged over 1 minute."
- Short: **"Billing pipeline freshness (p99)."**

If a service needs something the template doesn't cover (client-side latency, a different
percentile, only payloads under 1 kB), write the override and leave the rest implied.

### Objectives in Practice

Start from what users care about, not from what is easy to measure. The thing users care about is
often hard or impossible to measure, so you will use a proxy. If you start with whatever the
dashboard already has, the SLOs will be weaker. Working backward from the objective you want,
then picking indicators that support it, works better than picking indicators first and inventing
targets later.

## Key takeaways

- SLIs are the measurements (latency, error rate, throughput, availability, durability); SLOs
  are the targets you set on those measurements. You need a good SLI before an SLO means
  anything.
- Treat metrics as distributions, not averages. An average latency can stay flat all day while
  p95/p99 get 20× worse; alert on the tail you care about, not the mean.
- Standardize SLIs with reusable templates (availability, latency, throughput, durability,
  pipeline freshness). Once the house defaults are agreed, an individual SLI is just the service
  name plus any override.
- Latency is bounded and skewed (floor at 0 ms, timeout cap), so mean and median often diverge.
  Don't assume a bell curve: an "outlier" restart rule will fire too often on a heavy tail, or
  too rarely if timeouts hide the real hangs.
- Start from what users care about, then work backward to indicators, even if you have to
  approximate. Starting from what's easy to measure produces weaker SLOs.
- Prefer measuring what users actually experience over what's convenient to measure (client-side
  latency over server-side, for instance), and fall back to a proxy only when the real thing is
  out of reach.
- Availability as "fraction of well-formed requests that succeed" (yield) is the practical,
  achievable framing; 100% is off the table, so talk in nines instead (Google Compute Engine
  targets "three and a half nines", 99.95%).
- Some SLOs aren't yours to set (traffic volume is demand-driven), and the ones you do set can
  interact: pushing throughput up tends to push latency up too, up to a performance cliff.
- Publish your SLOs. An explicit, shared target beats letting users guess, since unstated
  expectations drift into either over-reliance (assuming more availability than you deliver) or
  under-reliance (assuming you're flakier than you are).
- SLA is an SLO plus a consequence for missing it. SRE doesn't own SLAs (they're a business
  decision), but does own keeping the service inside them and defining the SLIs they're measured
  against. Even without a formal SLA, unavailability still has real consequences (reputation,
  revenue), so tracking SLIs/SLOs is worthwhile regardless.

<!-- obsidian:end -->

## Interactive diagrams

| Diagram | Open |
| --- | --- |
| **Latency percentiles**: Figure 4-1 redrawn: p50 stays near 50 ms while p99 spikes to 10 s; an average would hide that | [View live](https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/latency-percentiles.html) · [source](diagrams/latency-percentiles.html) |
| **Mean vs median**: 100 requests, timeout at 1 s, in six steps: chopped tail, false outliers, hidden hangs | [View live](https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/statistical-fallacies.html) · [source](diagrams/statistical-fallacies.html) |

Open the HTML in a browser. Obsidian and GitHub markdown will not run the clicks.

## Questions / things to revisit

-
