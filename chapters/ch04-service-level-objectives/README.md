# Chapter 04 — Service Level Objectives

> Part II. Principles of *Site Reliability Engineering* (Google / O'Reilly).
>
> Notes last synced 2026-08-29.

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
  (p95 or p99) is moving a lot. See <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/latency-percentiles.html" target="_blank" rel="noopener noreferrer">Latency percentiles</a>.

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

See the six-step walkthrough: <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/statistical-fallacies.html" target="_blank" rel="noopener noreferrer">Mean vs median</a>.

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

##### Template: Pipeline freshness (batch or big data)

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

#### Defining Objectives

Write the SLO so a stranger can measure it the same way you do. Say *what* is measured and
*under which rules* it counts.

You do not have to repeat those rules in every SLO. The SLI template from the previous section
already holds them. That is why these two lines mean the same thing:

- Long: `99% (averaged over 1 minute) of Get RPC calls will complete in less than 100 ms
  (measured across all the backend servers).`
- Short: `99% of Get RPC calls will complete in less than 100 ms.`

The parentheses are not extra flavour. They are the house defaults (1-minute window, all backend
servers). Once the team agrees those defaults, the short line is enough. Anyone who needs the
missing details looks them up in the template.

**Why the second line is not a weaker promise**

Think of the template as a filled-in form. The long line writes every box. The short line writes
only the boxes that change (99%, Get RPC, 100 ms). The other boxes stay as they were. If you
changed a default (measure at the client, or average over 1 hour), you would say so. Silence
means "use the template."

**Why one target is not always enough: the shape of the curve**

A single SLO pins *one point* on the latency distribution (the set of all request times, from
fastest to slowest). Many different shapes can pass that one point.

Both services below meet `99% of Get RPCs finish in under 100 ms`. Users do not experience them
as the same product.

| | Service A (snappy typical) | Service B (slow typical) |
|---|---|---|
| 90 requests | 1 ms | 85 ms |
| 9 requests | 8 ms | 95 ms |
| 1 request | 90 ms | 99 ms |
| p90 | 1 ms | 85 ms |
| p99 | 90 ms | 99 ms |
| Passes `99% < 100 ms`? | yes | yes |
| Feels instant for most people? | yes | no |

Shape is important when the *typical* request and the *tail* request are different product
promises:

- **Search or a checkout button.** Sort the 100 request times from fastest to slowest. **p90**
  is the time of request number 90 in that list. 90 of 100 finished in that time or less. That
  is what most people feel when they click. **p99** is the time of request number 99. 99 of
  100 finished in that time or less. That is the unlucky 1%. If p99 is under 100 ms (the SLO
  still looks healthy) but p90 is already 85 ms, almost every click is slow. Users do not get
  the fast 1%. They get the slow 90. The product feels slow to everyone, not only to the
  unlucky one. One SLO at 100 ms cannot see this, because both the 85 ms clicks and the 99 ms
  click still pass.
- **A batch report.** Users will wait seconds for a nightly PDF. They do not need 90 of 100
  reports in 1 ms. They need the report to arrive before the morning meeting. So you watch the
  far tail: how late is the slow one?

  **What `p99` and `p99.9` mean.** The number after `p` is a percent. `p99` is the time that
  99 of 100 jobs finish in that time or less (1 in 100 is slower). `p99.9` is the time that
  999 of 1,000 jobs finish in that time or less (1 in 1,000 is slower).

  Example: 1,000 reports must be in inboxes by 08:00. 999 finish by 07:50. One finishes at
  10:00. p99 is still about 07:50 (only 1 in 100 is allowed to be later, and you have only 1
  late job in 1,000). p99.9 is 10:00: that is the one job that missed the meeting. If "almost
  nothing may be late" is the promise, p99.9 is the number you set a deadline on.

  Why more machines for p90 = 1 ms do not fix that SLO: the SLO you set is p99.9 (in by
  08:00). p90 is already a pass. 999 reports are in at 07:50. Buying CPUs so those 999 finish
  in 1 ms changes p90 only. It does not change p99.9. The 10:00 job is late for a different
  reason (a stuck worker, a huge file, a lock). Making the other 999 faster does not finish
  that job. The meeting still misses that one file. The SLO you set is still missed.
- **Two user populations.** The same API serves phones on slow networks and an internal admin
  tool on a fast office network. Phone requests cluster around a slower time. Admin requests
  cluster around a faster time. One 100 ms SLO is a compromise that fits neither group. Two
  targets (or two SLOs, one per client type) make both groups visible.
- **A regression that "still passes."** The common path is the code every request runs (auth,
  logging, a new feature check). Engineers add work there. Before: 90 of 100 requests finish
  in 1 ms. After: those same 90 finish in 80 ms. The one slow request is still under 100 ms,
  so p99 has not moved past the 100 ms SLO.

  If you only have one target (`99% under 100 ms`), that SLO is still true. No alert fires.
  90 of 100 clicks now take 80 ms instead of 1 ms.

  If you have the three targets below, the first one is now false: 90 of 100 requests are
  *not* under 1 ms. Monitoring marks that SLO as missed. **Page you** means it sends an
  on-call alert (phone, Slack, pager) to the engineer who is on duty, so someone looks at
  the change the same day, not after a week of tickets that say the site is slow.

That is why the book offers a *stack* of SLOs when shape matters:

- `90% of Get RPC calls will complete in less than 1 ms.`
- `99% of Get RPC calls will complete in less than 10 ms.`
- `99.9% of Get RPC calls will complete in less than 100 ms.`

Together they say: most requests finish in about 1 ms, almost all still finish in 10 ms, and
only a few may take as long as 100 ms. Service B above would miss the first two lines. See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/same-slo-two-shapes.html" target="_blank" rel="noopener noreferrer">Same SLO, two shapes</a>.

**Two classes of work on the same write**

A **heterogeneous workload** means mixed types of work on one API. Same `Set` call. Two
different customers.

An **RPC** (remote procedure call) is a function call that crosses the network: your code
calls `Set` here, another machine does the write there. `Set` is the write. `Get` would be
the read.

The two customers want different things from that same `Set`:

- **Throughput client:** a bulk pipeline. It wants many writes to finish per hour. Each write
  can take up to 1 second. The person is not waiting on a click.
- **Latency client:** a person waiting on a click (a phone, a form). Each *small* write must
  feel instant.

So you write two SLOs, not one:

- `95% of Set RPC calls from throughput clients will complete in less than 1 s.`
- `99% of Set RPC calls from latency clients, with payloads under 1 kB, will complete in
  less than 10 ms.`

**Payload** is the size of the data in the write. 1 kB is 1,024 bytes, about a short
paragraph of text. The second line only counts those small writes.

**Why two lines, not one number for every `Set`**

| If you write… | What happens |
|---|---|
| One SLO: every `Set` under 10 ms | The pipeline fails every day. A 10 MB nightly dump cannot finish in 10 ms. Your phone rings for that dump. The dump is fine. A slow click does not ring. |
| One SLO: every `Set` under 1 s | The phone can wait up to 1 second and the SLO stays green. No alert. The click feels broken. |
| One mixed bucket of all `Set`s | The pipeline may send millions of large writes. Combined p99 (the time 99 of 100 finish in or less) looks like those large writes. The 10 ms clicks disappear in the pile. |

**Why the latency line also says "payloads under 1 kB"**

A 10 MB write from a phone cannot finish in 10 ms either. If you put large writes in the
"fast click" SLO, that SLO is always red. The filter keeps that SLO about the writes a click
actually sends. Large writes are measured under a different rule, or under the throughput
SLO.

**How you tell the two classes apart**

A header, a client ID, a separate API key, or the payload-size filter. Monitoring must be
able to split the requests. If every `Set` looks the same, you cannot apply two SLOs.

**Why 95% on the pipeline and 99% on the click**

The pipeline can accept 5 slow writes in 100. A person clicking cannot accept 5 slow clicks
in 100. Only 1 in 100 clicks may miss 10 ms.

This is the same idea as the "two user populations" note above, now with numbers. See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/two-workload-classes.html" target="_blank" rel="noopener noreferrer">Two workload classes</a>.

**Do not require the SLO 100% of the time. Keep an error budget.**

An **error budget** is the allowed miss rate. Chapter 3 wrote it as `1 − SLO`. Same number,
now used as a daily working limit.

Example: the SLO is `99.9% of Sets succeed`. In 1,000 Sets, 999 must succeed. The leftover
**1 Set may fail**. That 1 is the budget.

A 100% SLO means the leftover is 0. That is both impossible and a bad product choice.

- **Unrealistic.** Something always fails: a deploy, a network blip, a bad payload. You will
  miss. Then the SLO is a lie.
- **It slows shipping.** Any failure is a miss. Teams freeze deploys. New features wait.
- **It gets expensive.** You buy extra regions, extra replicas, extra review gates, so nothing
  can fail. The success rate users already see (99.9%) does not change. The bill does.

So you write the miss on purpose. Then you **spend** it on purpose: a risky launch, a planned
Chubby outage, a week of faster deploys. The budget is the room to take those risks without
breaking the promise.

**Watch the spend on two clocks**

| Who | How often | Why |
|---|---|---|
| The team | Every day, or every week | You can still steer. Monday used 80 of 100 allowed misses. You slow deploys on Tuesday. |
| Upper management | Every month, or every quarter | Same number, zoomed out. The board does not need Monday. They need "this quarter we stayed inside 0.1%." |

Daily and weekly are not a different SLO. They are a finer grain of the same allowance. If you
only look once a quarter, the quarter is already over when you see you overspent.

**"An error budget is just an SLO for meeting other SLOs"**

That line packs two layers. Split them.

- **Product SLO** (the one users feel): `99.9% of Sets succeed.`
- **Budget SLO** (the one the team watches): `this week, the share of failed Sets will stay at
  or under 0.1%.`

The second line is itself an SLO. The SLI is "how much of the allowance have we used." The
target is "do not use more than 100% of the allowance." You are setting an objective on
whether you are still meeting the other objectives.

Worked week: 1,000,000 Sets. 0.1% of that is 1,000 allowed failures.

| Day | Failures | Budget left (of 1,000) |
|---|---|---|
| Monday | 800 | 200 |
| Tuesday | 50 | 150 |
| Wednesday–Sunday | 40 | 110 |

Monday did not break the product SLO by itself. It spent most of the week's budget SLO. Daily
tracking is what makes that visible on Monday, not in the quarterly report.

If the week ends at 0 left, you stop spending (freeze risky deploys) until the next window
refills the allowance. If the quarter ends with budget left, that leftover is what the Chubby
planned outage spends on purpose (see that case study above). Leftover is not a bonus. It is
unspent risk that other teams start to depend on.

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/error-budget.html" target="_blank" rel="noopener noreferrer">Error budget</a>.

#### Choosing Targets

**Do not copy today's number as the SLO.**

You still measure current performance. That tells you what the system can do today, and where
it is weak. The mistake is taking that dashboard number and publishing it as the target
without asking whether users need that number, and what it costs to keep hitting it.

**Heroic effort** means the number only holds because people do manual work the system cannot
do: two engineers restart stuck workers every night, or they babysit every deploy. That is
not a property of the product. It is overtime.

Example: tonight's dashboard says p99 (the time 99 of 100 requests finish in or less) is
10 ms. How? Two people restart workers at 02:00. You publish `99% of Sets finish in under
10 ms`. Now the night restarts are part of the promise. You cannot stop them without missing
the SLO. The only way out is a redesign that makes 10 ms true without the heroes. You locked
the current architecture in.

Reflect first. Ask: what do users need? If they are happy at 50 ms, write `99% under 50 ms`.
The night restarts can stop. The SLO stays green. You still know the system can do 10 ms
today. That fact is a merit, not a contract.

The other lock-in: today's p90 is 85 ms because the common path is fat (Service B in the
shape note above). If you copy 85 ms as the SLO, the slow typical click becomes the promise.
A later redesign that aims for 1 ms is "extra," because the SLO is already green.

**Have as few SLOs as possible.**

Pick just enough to cover the attributes users notice: usually availability, latency, and
maybe durability or freshness. Each extra SLO is another graph nobody watches.

**The meeting test.** An SLO earns its place if people use that number to decide "ship this
week, or wait." If a line has never changed a launch decision, it is not an SLO. You can
still put it on a dashboard.

Walk through one Friday meeting.

Today each click takes about 20 ms. Product wants to ship a photo filter this week. The
filter adds 80 ms to every click.

`20 ms + 80 ms = 100 ms` per click after the filter.

The written promise is `99% of small Sets finish in under 50 ms`. 100 is bigger than 50.
Most clicks would miss the promise.

So you say wait. That does not mean "SRE, make today's 20 ms click faster." The 20 ms
click already passes. The extra 80 ms is the problem.

**Wait** means: do not turn the filter on for everyone this week. The people who built
the filter (usually the product engineers) cut that 80 ms down. Example: they get it to
25 ms. Then `20 + 25 = 45`, which is under 50, and you can ship.

A **flag** is an on/off switch in the code. The filter can go to production switched
off for most users. A small test group can try it while the 80 ms is being cut. Most
users still get the 20 ms click, so the promise stays true.

The 50 ms line did the job: two teams disagreed, and the pre-agreed number picked the
next step. That is all "win an argument" means. It is not about being louder. It is
about pointing at a number everyone already accepted.

Now look at `CPU idle stays above 40%`. CPU idle is unused processor time. 40% idle means
the machines are busy only 60% of the time. Useful for capacity planning. In this meeting
nobody said "the filter is fine, idle is 55%." The launch did not move because of that
line. So it is not an SLO. Keep the graph. Drop it from the SLO list.

**User delight** fails the same test for a different reason. Delight has no shared ruler.
Two people will not agree that "delight is 80." You cannot send an on-call alert
(page you) for "less delighted." Keep delight as a product goal. Put an SLO on the pieces
you can count: the click was fast, the write stayed.

See
<a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/choosing-targets.html" target="_blank" rel="noopener noreferrer">Choosing targets</a>.

## Key takeaways

- SLIs are the measurements (latency, error rate, throughput, availability, durability); SLOs
  are the targets you set on those measurements. You need a good SLI before an SLO means
  anything.
- Treat metrics as distributions, not averages. An average latency can stay flat all day while
  p95 and p99 get 20× worse. Alert on the slow requests you care about, not the mean.
- Standardize SLIs with reusable templates (availability, latency, throughput, durability,
  pipeline freshness). Once the house defaults are agreed, an individual SLI is just the service
  name plus any override.
- Latency is bounded and skewed (floor at 0 ms, timeout cap), so mean and median often diverge.
  Don't assume a bell curve: an "outlier" restart rule will fire too often on a heavy tail, or
  too rarely if timeouts hide the real hangs.
- Start from what users care about, then work backward to indicators, even if you have to
  approximate. Starting from what's easy to measure produces weaker SLOs.
- A short SLO can omit the measurement rules once those rules live in the SLI template. The
  short line is not a weaker promise.
- One SLO pins one point on the distribution. Many shapes can pass `99% < 100 ms`. Use several
  targets (p90, p99, and p99.9) when both the typical request and the rare slow request matter
  to the product.
- The same `Set` RPC can need two SLOs when two client classes want different things. A bulk
  pipeline can wait 1 second. A person clicking cannot. Do not mix them into one bucket.
- Do not require an SLO 100% of the time. The leftover (`1 − SLO`) is the **error budget**:
  allowed misses you spend on purpose. Track the spend every day or every week so you can
  steer. A month or quarter rollup is for management. That "stay inside the allowance" line
  is itself an SLO on the other SLOs.
- Do not copy today's dashboard number as the SLO. Measure current performance to learn
  merits and limits. Publishing that number without reflection locks in today's architecture
  and any heroic effort that produced it. Keep as few SLOs as you can defend: if a line has
  never changed a "ship or wait" decision, drop it from the SLO list. "User delight" is not
  an SLO. It has no shared number.
- Prefer measuring what users actually experience over what's convenient to measure (client-side
  latency over server-side, for instance), and fall back to a proxy only when the real thing is
  out of reach.
- Availability as "fraction of well-formed requests that succeed" (yield) is the practical,
  achievable framing; 100% is off the table, so talk in nines instead (Google Compute Engine
  targets "three and a half nines", 99.95%).
- Some SLOs aren't yours to set: how many users arrive (traffic volume) is their choice, not a
  target you pick. The SLOs you do set can pull on each other. If you push the service to
  handle more requests per second, each request often gets slower. Past a load point (a
  performance cliff) latency jumps all at once, not a little at a time.
- Publish your SLOs. A written target is better than letting users guess. If you never say the
  number, people invent one: they assume you are more available than you are, or less available
  than you are.
- SLA is an SLO plus a consequence for missing it. SRE doesn't own SLAs (they're a business
  decision), but does own keeping the service inside them and defining the SLIs they're measured
  against. Even without a formal SLA, unavailability still has real consequences (reputation,
  revenue), so tracking SLIs/SLOs is worthwhile regardless.

<!-- obsidian:end -->

## Interactive diagrams

| Diagram | Open |
| --- | --- |
| **Latency percentiles**: Figure 4-1 redrawn: p50 stays near 50 ms while p99 spikes to 10 s; an average would hide that | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/latency-percentiles.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Mean vs median**: 100 requests, timeout at 1 s, in six steps: chopped tail, false outliers, hidden hangs | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/statistical-fallacies.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Same SLO, two shapes**: both services pass `99% < 100 ms`; only one has p90 at 1 ms | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/same-slo-two-shapes.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Two workload classes**: same `Set` RPC; pipeline 95% under 1 s, small clicks 99% under 10 ms | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/two-workload-classes.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Error budget**: 99.9% succeed means 1 of 1,000 may fail; that leftover is the spend you track | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/error-budget.html" target="_blank" rel="noopener noreferrer">Open</a> |
| **Choosing targets**: do not copy today's 10 ms if heroes produce it; keep only SLOs that change a ship-or-wait decision | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/choosing-targets.html" target="_blank" rel="noopener noreferrer">Open</a> |

Open the HTML in a browser. Obsidian and GitHub markdown will not run the clicks.

## Questions / things to revisit

-
