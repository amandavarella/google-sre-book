# Chapter 06 — Monitoring Distributed Systems

> Part II. Principles of *Site Reliability Engineering* (Google / O'Reilly).

---

## Black-Box Versus White-Box

**Definitions** (from the book's Definitions section):

- **White-box monitoring:** monitoring based on numbers that the system reports about its
  own insides. Sources: logs, interfaces that report what a running program is doing (for
  example the Java Virtual Machine Profiling Interface), or an HTTP handler (a web address
  on the server) that returns internal statistics.
- **Black-box monitoring:** testing the behavior you can see from outside, the way a user
  would see it.

**How Google uses them:** heavy use of white-box monitoring, with modest but critical use
of black-box monitoring.

**Black-box shows symptoms.** A **symptom** is what is broken; a **cause** is why. Black-box
monitoring shows problems that are happening now, not predicted ones: "the system is not
working correctly, right now."

**White-box looks inside.** It needs **instrumentation** (code that reports what the
system is doing), such as logs or HTTP endpoints. Because it sees inside, it can catch:

- problems that are about to happen
- failures hidden by **retries** (the request failed, was tried again, and then worked, so
  the user never saw the failure)

**One person's symptom is another person's cause.** In a system built in layers, the same
problem looks different from each layer. Example: a database is slow.

- For the database SRE, slow database reads are a **symptom**.
- For the frontend SRE watching a slow website, the same slow reads are a **cause**.

So white-box monitoring can be about symptoms or about causes, depending on how much your
white-box data tells you.

**White-box is essential for debugging.** Example: web servers seem slow on requests that
use the database a lot. You need two numbers:

- how fast the web server thinks the database is
- how fast the database thinks it is

Without both, you cannot tell a slow database apart from a network problem between the web
server and the database.

**Black-box is useful for paging.** A page (an on-call alert) from black-box monitoring
means the problem is already happening and already hurting users. That forces you to
interrupt a human only for real problems. But for problems that have not happened yet and
are about to, black-box monitoring is fairly useless.

## The Four Golden Signals

The four golden signals are **latency**, **traffic**, **errors**, and **saturation**.
If you can measure only four metrics of your user-facing system, the book says to pick
these four.

### Latency

- **Latency** is the time it takes to answer a request.
- Keep successful requests and failed requests apart.
- Example: an **HTTP 500** (the server's "I failed" response) caused by losing the
  connection to a database can come back very fast. If you mix those fast failures into
  the overall latency, the numbers look better than what successful users get.
- A slow error is even worse than a fast error. So track the latency of errors too. Do
  not just filter errors out.

### Traffic

- **Traffic** is how much demand is placed on your system. Measure it with one high-level
  number that fits the kind of system.
- Web service: HTTP requests per second, maybe split by kind of request (for example,
  static pages compared with dynamic pages).
- Audio streaming system: network **I/O rate** (how much data goes in and out per second),
  or the number of sessions open at the same time.
- Key-value storage system: transactions and reads per second.

### Errors

**Errors** are the rate of requests that fail. A request can fail in three ways:

- **Explicitly:** the response says it failed, for example an HTTP 500.
- **Implicitly:** the response says it worked (an **HTTP 200**, the "OK" response) but
  the content is wrong.
- **By policy:** you promised one-second responses, so any request that takes longer than
  one second counts as an error, even if the answer is right.

Catching each kind takes different monitoring:

- If the response codes cannot express every kind of failure, you may need extra internal
  protocols to track partial failures.
- HTTP 500s counted at the **load balancer** (the front door that spreads requests across
  servers) catch requests that failed completely.
- Only **end-to-end tests** (a test that acts like a user and checks the real answer) can
  catch a 200 with the wrong content.

### Saturation

- **Saturation** is how "full" your service is. Watch the resource that runs out first: in
  a system limited by memory, show memory; in a system limited by I/O, show I/O.
- Many systems slow down before they reach 100% use, so you need a utilization target
  below 100%.
- In complex systems, add a higher-level load question: could the service handle double
  its traffic, only 10% more, or even less than it gets now?
- Very simple services can use one fixed limit measured in a **load test** (a test that
  pushes traffic until the service struggles). This works only when requests do not change
  in complexity (for example "give me a **nonce**", a one-time random value, or "give me
  the next unique number") and the configuration rarely changes.
- Most services instead need indirect signals that have a known upper limit, such as CPU
  use or network bandwidth.
- Rising latency is often the **first warning** of saturation. Watching the **99th
  percentile** response time (the time that 99 of every 100 requests finished in or less;
  see the <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/latency-percentiles.html" target="_blank" rel="noopener noreferrer">Chapter 4 latency percentiles diagram</a>)
  over a short window, such as one minute, can show saturation very early.
- Saturation also includes predictions: "your database will fill its hard drive in 4 hours."

### Paging on the four signals

Measure all four signals. **Page** a human (send an on-call alert) when one of them is a
problem, or, for saturation, nearly a problem. Then the book says your service is at least
decently covered by monitoring.

## Worrying About Your Tail (or, Instrumentation and Performance)

- A **tail** is the slow end of the latency distribution: the few requests that take far
  longer than most.
- Example from the book: a web service has an **average latency of 100 ms** at **1,000
  requests per second**. **1% of requests** (10 per second) can easily take **5 seconds**.
  The average hides them.
  - The other **99%** (990 per second) are fast.
  - The **99th percentile** is the line between the two groups: 99 of every 100 requests
    finish fast, and the slowest 1 of every 100 takes 5 seconds. The 5-second requests sit
    just past the 99th percentile.
- Footnote 23: if 1% of requests are 50 times the average, the other requests are about
  twice as fast as the average (here, about 50 ms). If you do not measure the distribution,
  believing most requests are near the average is just hope.
- To build one web page, the **frontend** (the server that builds the web page) sends a
  request to several **backends**. A backend is a service that answers one part of the
  web page. The book's case: each backend is a web service like the one above (100 ms
  average, 1% of requests at 5 seconds).
- The frontend can show the web page only after every backend answers. So one slow answer
  makes the whole web page slow.
- Because of this, the **99th percentile** of one backend (the time 99 of 100 requests
  finish in or less) can easily become the **median** (the middle value) response time of
  the frontend.

Step through it in the <a href="https://amandavarella.github.io/google-sre-book/chapters/ch06-monitoring-distributed-systems/diagrams/tail-latency.html" target="_blank" rel="noopener noreferrer">Worrying about your tail diagram</a>.

## Choosing an Appropriate Resolution for Measurements

**Resolution** (also called **granularity**) is how often you take a measurement, or how
long a window each data point covers. Different parts of a system need different
resolutions.

- **CPU load:** if each data point is the CPU load over one minute, you will not see
  spikes, even quite long ones, that cause high tail latency (the slow end of request
  times, from the [section above](#worrying-about-your-tail-or-instrumentation-and-performance)).
  The one-minute average mixes a short spike with the normal seconds around it, so the
  spike does not show.
- **Checking for success:** a web service aims for **99.9% uptime per year** (uptime: the
  time the service is working), which is
  about **9 hours of total downtime per year**. A **probe** (a test request that checks for
  an HTTP 200, the "OK" response) sent more than once or twice a minute is probably more
  often than needed.
- **Hard drive fullness:** for a service aiming at 99.9% availability (the share of time
  the service works), checking more often
  than once every 1 to 2 minutes is probably unnecessary.

**Fine resolution costs money.**

- Measuring CPU load every second can give interesting data. But data that frequent can be
  very expensive to collect, store, and analyze.
- When you need high resolution but do not need the result with very little delay, the
  book says you can cut the cost: the server **samples** (measures) itself often, and an
  outside monitoring system collects and **aggregates** (combines) the results over time
  or across servers.

The book's example:

1. Record the current CPU use every second.
2. Keep one counter per **bucket**, a range of 5% (0 to 5%, 5 to 10%, and so on up to
   100%). Every second, add 1 to the counter for the range that second's reading fell in.
3. Every minute, send the counters on and combine them.

This shows brief CPU hotspots (short spikes of high CPU use) without the very high cost of
collecting and keeping every per-second reading. (A minute of readings becomes one set of
counters that adds up to 60, instead of 60 separate values.)

## Keeping it simple

It can be tempting to put other ways of inspecting a complex system into the monitoring
system too:

- detailed system **profiling** (measuring where a program spends its time and memory)
- **single-process debugging** (looking inside one running program to find a bug)
- tracking details about exceptions or crashes
- **load testing** (pushing traffic at a service until it struggles)
- collecting and analyzing logs
- **traffic inspection** (looking at the actual requests going over the network)

Most of these have things in common with basic monitoring. But blending too many of them
into one system makes it overly complex and **fragile** (it breaks easily).

The book's better strategy, as in much of software engineering: keep them as separate
systems, joined by clear, simple, **loosely coupled** connection points. Loosely coupled
means one system can change inside without breaking the others, as long as the connection
point stays the same.

- Example: one system offers a **web API** (a fixed web address another program calls to
  get data). The other system pulls summary data from it, in a format that can stay the
  same for a long time.

## Questions to avoid false positives and pager burnout

When you write a rule for monitoring and alerting, the book says these questions help you
avoid **false positives** (an alert fires, but nothing is wrong for users) and **pager
burnout** (on-call people worn out by too many alerts). A **page** is an on-call alert.

1. **Does this rule catch something that would otherwise go unnoticed?** And is that
   condition urgent, **actionable** (a person can do something about it), and visible to
   users now or soon?
   - Footnote 25: "soon" includes running with **zero redundancy** (N + 0: no spare, so
     one more failure hits users) and parts of the service that are nearly full.
2. **Will I ever ignore this alert because I know it is harmless?** When and why would I
   ignore it, and how can I avoid that situation?
3. **Does this alert definitely mean users are being hurt?** Some cases can be detected
   where users are not affected, and those should be filtered out. Examples: **drained
   traffic** (users were moved away from that part of the service on purpose) and test
   deployments.
4. **Can I act on this alert?**
   - Is the action urgent, or could it wait until morning?
   - Could the action be safely automated?
   - Is it a long-term fix, or only a short-term **workaround** (a patch that hides the
     problem for now)?
5. **Are other people being paged for the same issue?** If so, at least one of the pages
   is unnecessary.

## Philosophy on pages and pagers

The questions above come from one basic view of pages (on-call alerts) and pagers:

- **Every page needs a sense of urgency.** When the pager goes off, I should be able to
  react urgently. A person can do that only a few times a day before **fatigue** (being
  too tired to react well) sets in.
- **Every page should be actionable.** The person paged can do something about it.
- **Every page response should need intelligence.** If a page only needs a **robotic
  response** (the same fixed steps every time, with no thinking), it should not be a page.
- **Pages should be about a novel problem**: a problem or event not seen before.

## Key takeaways

- White-box monitoring reads the system's insides (logs, internal stats). Black-box
  monitoring tests from outside, as a user would. Google uses white-box heavily and
  black-box for a few critical checks.
- Black-box shows problems happening now (good for pages). White-box also catches problems
  about to happen and failures hidden by retries, and is essential for debugging.
- The four golden signals: latency, traffic, errors, saturation. Measure these first.
- Track the latency of failed requests separately from successful ones. A slow error is
  worse than a fast one.
- Errors can be explicit (HTTP 500), implicit (HTTP 200 with wrong content), or by policy
  (slower than the promised one second).
- Saturation means the most limited resource; rising 99th percentile latency is an early
  warning.
- Page a human when a signal is a problem (saturation: nearly a problem).
- An average can hide a slow tail: 100 ms average with 1% of requests at 5 s. When a web
  page needs an answer from several backends, one backend's 99th percentile can become the
  web page's median.
- Pick a resolution for each measurement. One-minute CPU averages hide spikes. For a 99.9%
  target, probing more than once or twice a minute is probably too often.
- To see short spikes cheaply, sample every second on the server into 5% buckets and
  aggregate every minute.
- Keep monitoring separate from profiling, debugging, crash tracking, load testing, logs,
  and traffic inspection. Join the systems through simple, stable connection points, such
  as a web API for summary data.
- Before adding an alert rule, ask: does it catch something unnoticed, urgent, actionable,
  and visible to users? Would I ever ignore it? Are users definitely hurt? Can I act on
  it now? Is someone else already paged for it?
- Every page should be urgent, actionable, need human thinking, and be about a new problem.

## Interactive diagrams

| Diagram | Open |
| --- | --- |
| **Worrying about your tail**: average 100 ms, 10 of 1,000 requests take 5 s; with 1, 10, then 70 backends, the slow share of web pages grows until the median web page takes 5 s | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch06-monitoring-distributed-systems/diagrams/tail-latency.html" target="_blank" rel="noopener noreferrer">Open</a> |

Open the HTML in a browser. GitHub markdown will not run the clicks.

## Questions / things to revisit

- 
