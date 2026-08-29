# Chapter 04 — Service Level Objectives

> Part II. Principles of *Site Reliability Engineering* (Google / O'Reilly).
>
> Notes last synced 2026-08-29.

---

<!-- obsidian:start -->

## Service Level Terminology

The book splits one overloaded word (people say "SLA" for everything) into three: the
measurement (SLI), the target (SLO), and the contract with a consequence (SLA). These notes
keep that split.

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

### SLI vs SLO

The two names sound alike. They are not the same thing.

| | SLI | SLO |
|---|---|---|
| Full name | Service level **indicator** | Service level **objective** |
| What it is | The measurement. The ruler. | The target on that measurement. The mark on the ruler. |
| Question it answers | "What number are we reading?" | "What number did we promise?" |
| Example shape | a count, a rate, a percentile | `SLI` compared to a number |

You cannot have an SLO without an SLI. The SLO is always:

`the SLI` + `a comparison` + `a number`

Example: SLI = share of well-formed write requests that succeed. Comparison = at least.
Number = 99.9%. The SLO is `at least 99.9% of well-formed write requests succeed`.

The SLI can be green or red only after you attach an SLO. A dashboard that says "success
rate is 99.7%" is an SLI reading. It is not a miss until someone wrote "we promised 99.9%."

One SLI can feed more than one SLO (two marks on the same ruler). Some SLIs never get an
SLO: incoming QPS is a real measurement, but users choose that number, so you do not
promise it.

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/sli-vs-slo.html" target="_blank" rel="noopener noreferrer">SLI vs SLO</a>
(pairs, then one ruler with two marks).

### The Global Chubby Planned Outage

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

## Indicators in Practice

### What Do You and Your Users Care About?

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

### Collecting Indicators

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

### Aggregation

Aggregate raw measurements carefully. Most metrics are distributions, not averages.

- For a latency SLI, some requests finish quickly and others take much longer. A simple average
  hides those tail latencies, and hides changes in them.
- Figure 4-1: a typical request is served in about 50 ms, but 5% of requests are 20 times slower.
  If you monitor and alert only on average latency, the day looks unchanged, even while the tail
  (p95 or p99) is moving a lot. See <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/latency-percentiles.html" target="_blank" rel="noopener noreferrer">Latency percentiles</a>.

#### A Note on Statistical Fallacies

Percentiles are usually more useful than the mean. They show the long tail. Computer systems
chop that tail: nothing finishes in less than 0 ms, and a 1 s timeout means no *success* can
be slower than 1 s. Mean and median need not be close.

Do not assume a bell curve. A "restart on 3σ outlier" rule will fire too often on ordinary
slow-but-ok requests (false outliers), and too rarely on hangs the timeout recorded as
exactly 1 s.

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/statistical-fallacies.html" target="_blank" rel="noopener noreferrer">Mean vs median</a>
(100 requests, chopped tail, false outliers, hidden hangs).

### Standardize Indicators

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

#### Template: Availability (serving)

Proportion of well-formed requests that succeed (not 5xx). Client errors (4xx) are excluded, they
are not the service failing.

- Full: "Proportion of well-formed HTTP GET requests to the Shakespeare search frontend that are
  not 5xx, measured at the server, averaged over 1 minute, across all tasks in the cluster."
- Once the template exists: **"Availability of Shakespeare search."**

#### Template: Latency (serving)

Time to last byte, as a distribution (p50 and p99), for HTTP GETs from black-box probing.

- Full: "99th percentile of time-to-last-byte for HTTP GET requests to Shakespeare search from
  black-box monitoring jobs, measured every 10 seconds, averaged over 1 minute."
- Short: **"Search frontend latency (p99)."**
- Payoff in an SLO: you can write `99% of Get RPC calls will complete in less than 100 ms`
  instead of `99% (averaged over 1 minute) of Get RPC calls will complete in less than 100 ms
  (measured across all the backend servers)`. Same meaning, because the parentheses are already
  in the template.

#### Template: Throughput (serving)

Successful requests per second, using the same request filter as availability.

- Short: **"Search QPS (successful)."**
- Note: incoming demand is not an SLO you set, but successful throughput is still a useful SLI
  for capacity and cliffs.

#### Template: Durability (storage)

Proportion of records written that can still be read later (is the data still there when we need
it).

- Full: "Proportion of objects successfully written to the photo store in the last 365 days that
  can be retrieved with a matching checksum, measured by a weekly audit job."
- Short: **"Photo store durability."**

#### Template: Pipeline freshness (batch or big data)

End-to-end time from ingestion to a completed, correct output record, plus records processed per
minute.

- Full: "99th percentile of time from a row landing in the ingest topic to it appearing in the
  serving table, excluding retries after a known poison-pill, averaged over 1 minute."
- Short: **"Billing pipeline freshness (p99)."**

If a service needs something the template doesn't cover (client-side latency, a different
percentile, only payloads under 1 kB), write the override and leave the rest implied.

## Objectives in Practice

Start from what users care about, not from what is easy to measure. The thing users care about is
often hard or impossible to measure, so you will use a proxy. If you start with whatever the
dashboard already has, the SLOs will be weaker. Working backward from the objective you want,
then picking indicators that support it, works better than picking indicators first and inventing
targets later.

### Defining Objectives

Write the SLO so a stranger can measure it the same way you do. Say *what* is measured and
*under which rules* it counts.

You do not have to repeat those rules in every SLO. The SLI template from the previous section
already holds them. That is why these two lines mean the same thing:

- Long: `99% (averaged over 1 minute) of Get RPC calls will complete in less than 100 ms
  (measured across all the backend servers).`
- Short: `99% of Get RPC calls will complete in less than 100 ms.`

The parentheses are the house defaults (1-minute window, all backend servers). Once those
are agreed, the short line is enough. Silence means "use the template." The short line is
not a weaker promise.

**One target is not always enough.** A single SLO pins one point on the distribution. Two
shapes can both pass `99% under 100 ms` while most users feel something different. **p90**
is the time 90 of 100 finished in or less (what most people feel). **p99** is the time 99
of 100 finished in or less. When both the typical click and the rare slow one matter, write
a stack:

- `90% of Get RPC calls will complete in less than 1 ms.`
- `99% of Get RPC calls will complete in less than 10 ms.`
- `99.9% of Get RPC calls will complete in less than 100 ms.`

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/same-slo-two-shapes.html" target="_blank" rel="noopener noreferrer">Same SLO, two shapes</a>.

**Two classes of work on the same write.** A **heterogeneous workload** is mixed types of
work on one API. An **RPC** is a function call that crosses the network (`Set` = write).
A pipeline can wait 1 s. A person clicking cannot. **Payload** is the size of the write;
the click SLO only counts small ones (under 1 kB). Write two SLOs, and split the traffic
so monitoring can tell them apart:

- `95% of Set RPC calls from throughput clients will complete in less than 1 s.`
- `99% of Set RPC calls from latency clients, with payloads under 1 kB, will complete in
  less than 10 ms.`

One number for every `Set` either pages you for a fine dump, or hides the slow clicks.
See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/two-workload-classes.html" target="_blank" rel="noopener noreferrer">Two workload classes</a>.

**Do not require the SLO 100% of the time.** An **error budget** is the allowed miss rate
(`1 − SLO`). Example: `99.9% of Sets succeed` means 1 of 1,000 may fail. That leftover is
what you spend on purpose (a launch, a planned Chubby outage). A 100% SLO sets the leftover
to 0: unrealistic, it freezes deploys, and it gets expensive. Watch the spend every day or
week so you can still steer. A month or quarter rollup is for management. That "stay inside
the allowance" line is itself an SLO on the other SLOs.

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/error-budget.html" target="_blank" rel="noopener noreferrer">Error budget</a>.

### Choosing Targets

#### Don't pick a target based on current performance

Measure current performance to learn merits and limits. Do not publish tonight's dashboard
number as the SLO. **Heroic effort** means the number only holds because people do manual
work (restart workers at 02:00). Copy 10 ms and those night restarts become the promise.

Ask what users need. If they are happy at 50 ms, write that. Today's 10 ms stays a merit,
not a contract.

#### Have as few SLOs as possible

Pick just enough to cover what users notice (usually availability, latency, maybe
durability or freshness).

**The meeting test:** keep a line only if people use it to decide ship or wait. **Wait**
means do not turn the filter on this week. Product cuts the extra 80 ms. It is not "SRE,
make the 20 ms click faster." **CPU idle** is a capacity graph. It did not change ship
or wait, so it is not an SLO. **User delight** has no shared ruler, so it is not an SLO
either.

The SLO is a **lever**: too tight wastes heroic work, too loose lets a slow product stay
green. Pick the number by what users need.

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/choosing-targets.html" target="_blank" rel="noopener noreferrer">Choosing targets</a>
(Friday meeting, then the lever).

### Control Measures

SLIs and SLOs are the two numbers in a **control loop** (like a thermostat: reading vs
setpoint):

1. Measure the SLI.
2. Compare it to the SLO. Decide if action is needed.
3. If yes, find what would meet the mark (a hypothesis, then a test).
4. Act. Then go back to step 1.

Without the SLO you would not know **whether** to act (add servers today at all) or
**when** (at 15:00, before the miss). The same 28 → 45 ms path is a fire at a 50 ms mark
and not a fire at a 200 ms mark.

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/control-loop.html" target="_blank" rel="noopener noreferrer">Control loop</a>.

### SLOs Set Expectations

Publishing the SLOs tells users what the product will do, so they can decide "is this
right for us?" **Availability** is "can I use it *now*?" **Durability** is "will the file
still be there *later*?" You rarely get always-up, never-lose-a-file, and cheap. The
published mix is the offer. Same cheap + keep-the-file + sometimes-down store: a photo
app says no, an archive says yes.

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/slo-expectations.html" target="_blank" rel="noopener noreferrer">SLOs set expectations</a>.

#### Keep a safety margin

Write two marks on the same SLI. The **advertised SLO** is what users see. The
**internal SLO** is tighter. You act in the gap, before users see a miss (advertised
100 ms, internal 80 ms).

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/safety-margin.html" target="_blank" rel="noopener noreferrer">Safety margin</a>.

#### Don't overachieve

Users depend on what they actually get, not the written SLO. If you run much better than
you advertised, they treat that as the real promise, which is why Chubby takes planned
outages (see [The Global Chubby Planned Outage](#the-global-chubby-planned-outage) above).

The SLO also picks the next week of people. Internal red or budget almost gone: invest in
faster, more available, or more resilient. Green and budget left: pay down **technical
debt** (a shortcut that makes the next change harder), ship features, or start another
product. Do not spend a green week turning 28 ms into 20 ms.

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/invest-where.html" target="_blank" rel="noopener noreferrer">Invest where</a>.

## Key takeaways

- SLI = the measurement. SLO = the mark on that measurement (`SLI` + a comparison + a number).
  SLA = an SLO plus a consequence for missing it.
- Treat latency as a distribution, not an average. Percentiles show the tail. Do not assume
  a bell curve (see [Mean vs median](https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/statistical-fallacies.html)).
- Start from what users care about, then pick indicators. Standardize SLIs with house
  templates so a short SLO is not a weaker promise.
- One SLO pins one point. Use a stack, or two SLOs on the same `Set`, when typical and tail
  (or pipeline and click) are different promises.
- An error budget is `1 − SLO`. Spend it on purpose. Watch it daily or weekly.
- Do not copy today's dashboard number. Keep only SLOs that change ship or wait. Too tight
  wastes heroes. Too loose lets a slow product stay green.
- Control loop: measure, compare, decide, act. Without the mark you do not know whether or
  when. Publish the mix so buyers can choose. Keep a tighter internal SLO. Do not
  overachieve (Chubby). The SLO also picks the next week of people.

<!-- obsidian:end -->

## Interactive diagrams

| Diagram | Open |
| --- | --- |
| **SLI vs SLO**: the ruler (what you count) and the mark (what you promised); four pairs, then one ruler with two marks | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/sli-vs-slo.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Latency percentiles**: Figure 4-1 redrawn: p50 stays near 50 ms while p99 spikes to 10 s; an average would hide that | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/latency-percentiles.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Mean vs median**: 100 requests, timeout at 1 s, in six steps: chopped tail, false outliers, hidden hangs | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/statistical-fallacies.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Same SLO, two shapes**: both services pass `99% < 100 ms`; only one has p90 at 1 ms | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/same-slo-two-shapes.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Two workload classes**: same `Set` RPC; pipeline 95% under 1 s, small clicks 99% under 10 ms | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/two-workload-classes.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Error budget**: 99.9% succeed means 1 of 1,000 may fail; that leftover is the spend you track | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/error-budget.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Choosing targets**: do not copy today's 10 ms if heroes produce it; keep only SLOs that change ship or wait; last step is the lever (too tight, just enough, too loose) | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/choosing-targets.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Control loop**: measure the SLI, compare to the SLO, find an action, act; without the mark, 28 ms → 45 ms has no deadline | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/control-loop.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **SLOs set expectations**: same offer (cheap, keep the file, sometimes down); photo app says no, archive says yes | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/slo-expectations.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Safety margin**: advertised 100 ms, internal 80 ms; you act in the 20 ms gap before users see a miss | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/safety-margin.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Invest where**: SLO red or budget gone, fix the service; green and budget left, features and debt | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/invest-where.html" target="_blank" rel="noopener noreferrer">Open</a> |

Open the HTML in a browser. Obsidian and GitHub markdown will not run the clicks.

## Questions / things to revisit

-
