# Chapter 06 — Monitoring Distributed Systems

> Part II. Principles of *Site Reliability Engineering* (Google / O'Reilly).

---

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

## Key takeaways

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

## Interactive diagrams

| Diagram | Open |
| --- | --- |
| **Worrying about your tail**: average 100 ms, 10 of 1,000 requests take 5 s; with 1, 10, then 70 backends, the slow share of web pages grows until the median web page takes 5 s | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch06-monitoring-distributed-systems/diagrams/tail-latency.html" target="_blank" rel="noopener noreferrer">Open</a> |

Open the HTML in a browser. GitHub markdown will not run the clicks.

## Questions / things to revisit

- 
