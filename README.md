# Google SRE Book — Learning Notes

My notes, takeaways and **interactive diagrams** while working through
[*Site Reliability Engineering*](https://sre.google/books/) (Beyer, Jones, Petoff & Murphy — O'Reilly, 2016).

📖 **[Browse the interactive version →](https://amandavarella.github.io/google-sre-book/)**

## How this repo is organised

```
README.md                         you are here
index.html                        GitHub Pages landing page
AGENTS.md                         conventions for agents working in this repo
chapters/
  chNN-slug/
    README.md                     notes for that chapter
    diagrams/                     interactive HTML diagrams, one file each
```

Every diagram is a single self-contained HTML file, so it can be opened straight from disk
or viewed live on GitHub Pages.

## Chapters

| # | Chapter | Notes | Diagrams |
| --- | --- | --- | --- |
| 1 | [Introduction](https://sre.google/sre-book/introduction/) | [notes](chapters/ch01-introduction/README.md) | — |
| 2 | [The Production Environment at Google, from the Viewpoint of an SRE](https://sre.google/sre-book/production-environment/) | [notes](chapters/ch02-production-environment/README.md) | 1 |
| 3 | [Embracing Risk](https://sre.google/sre-book/embracing-risk/) | [notes](chapters/ch03-embracing-risk/README.md) | — |
| 4 | [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) | [notes](chapters/ch04-service-level-objectives/README.md) | 6 |
| 5 | [Eliminating Toil](https://sre.google/sre-book/eliminating-toil/) | [notes](chapters/ch05-eliminating-toil/README.md) | — |
| 6 | [Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) | [notes](chapters/ch06-monitoring-distributed-systems/README.md) | — |
| 7 | [The Evolution of Automation at Google](https://sre.google/sre-book/automation-at-google/) | [notes](chapters/ch07-evolution-of-automation/README.md) | — |
| 8 | [Release Engineering](https://sre.google/sre-book/release-engineering/) | [notes](chapters/ch08-release-engineering/README.md) | — |
| 9 | [Simplicity](https://sre.google/sre-book/simplicity/) | [notes](chapters/ch09-simplicity/README.md) | — |
| 10 | [Practical Alerting from Time-Series Data](https://sre.google/sre-book/practical-alerting/) | [notes](chapters/ch10-practical-alerting/README.md) | — |
| 11 | [Being On-Call](https://sre.google/sre-book/being-on-call/) | [notes](chapters/ch11-being-on-call/README.md) | — |
| 12 | [Effective Troubleshooting](https://sre.google/sre-book/effective-troubleshooting/) | [notes](chapters/ch12-effective-troubleshooting/README.md) | — |
| 13 | [Emergency Response](https://sre.google/sre-book/emergency-response/) | [notes](chapters/ch13-emergency-response/README.md) | — |
| 14 | [Managing Incidents](https://sre.google/sre-book/managing-incidents/) | [notes](chapters/ch14-managing-incidents/README.md) | — |
| 15 | [Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/) | [notes](chapters/ch15-postmortem-culture/README.md) | — |
| 16 | [Tracking Outages](https://sre.google/sre-book/tracking-outages/) | [notes](chapters/ch16-tracking-outages/README.md) | — |
| 17 | [Testing for Reliability](https://sre.google/sre-book/testing-reliability/) | [notes](chapters/ch17-testing-for-reliability/README.md) | — |
| 18 | [Software Engineering in SRE](https://sre.google/sre-book/software-engineering-in-sre/) | [notes](chapters/ch18-software-engineering-in-sre/README.md) | — |
| 19 | [Load Balancing at the Frontend](https://sre.google/sre-book/load-balancing-frontend/) | [notes](chapters/ch19-load-balancing-frontend/README.md) | — |
| 20 | [Load Balancing in the Datacenter](https://sre.google/sre-book/load-balancing-datacenter/) | [notes](chapters/ch20-load-balancing-datacenter/README.md) | — |
| 21 | [Handling Overload](https://sre.google/sre-book/handling-overload/) | [notes](chapters/ch21-handling-overload/README.md) | — |
| 22 | [Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/) | [notes](chapters/ch22-cascading-failures/README.md) | — |
| 23 | [Managing Critical State: Distributed Consensus for Reliability](https://sre.google/sre-book/managing-critical-state/) | [notes](chapters/ch23-distributed-consensus/README.md) | — |
| 24 | [Distributed Periodic Scheduling with Cron](https://sre.google/sre-book/distributed-periodic-scheduling/) | [notes](chapters/ch24-distributed-cron/README.md) | — |
| 25 | [Data Processing Pipelines](https://sre.google/sre-book/data-processing-pipelines/) | [notes](chapters/ch25-data-processing-pipelines/README.md) | — |
| 26 | [Data Integrity: What You Read Is What You Write](https://sre.google/sre-book/data-integrity/) | [notes](chapters/ch26-data-integrity/README.md) | — |
| 27 | [Reliable Product Launches at Scale](https://sre.google/sre-book/reliable-product-launches/) | [notes](chapters/ch27-reliable-product-launches/README.md) | — |
| 28 | [Accelerating SREs to On-Call and Beyond](https://sre.google/sre-book/accelerating-sre-on-call/) | [notes](chapters/ch28-accelerating-sres/README.md) | — |
| 29 | [Dealing with Interrupts](https://sre.google/sre-book/dealing-with-interrupts/) | [notes](chapters/ch29-dealing-with-interrupts/README.md) | — |
| 30 | [Embedding an SRE to Recover from Operational Overload](https://sre.google/sre-book/operational-overload/) | [notes](chapters/ch30-embedding-an-sre/README.md) | — |
| 31 | [Communication and Collaboration in SRE](https://sre.google/sre-book/communication-and-collaboration/) | [notes](chapters/ch31-communication-collaboration/README.md) | — |
| 32 | [The Evolving SRE Engagement Model](https://sre.google/sre-book/evolving-sre-engagement-model/) | [notes](chapters/ch32-sre-engagement-model/README.md) | — |
| 33 | [Lessons Learned from Other Industries](https://sre.google/sre-book/lessons-learned/) | [notes](chapters/ch33-lessons-from-other-industries/README.md) | — |
| 34 | [Conclusion](https://sre.google/sre-book/conclusion/) | [notes](chapters/ch34-conclusion/README.md) | — |

Parts follow the book: **I. Introduction** (1–2) · **II. Principles** (3–9) ·
**III. Practices** (10–27) · **IV. Management** (28–32) · **V. Conclusions** (33–34).

## Interactive diagrams

| Chapter | Diagram | View |
| --- | --- | --- |
| 2 | Life of a Request | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch02-production-environment/diagrams/life-of-a-request.html" target="_blank" rel="noopener noreferrer">open</a> |
| 4 | Latency percentiles | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/latency-percentiles.html" target="_blank" rel="noopener noreferrer">open</a> |
| 4 | Mean vs median | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/statistical-fallacies.html" target="_blank" rel="noopener noreferrer">open</a> |
| 4 | Same SLO, two shapes | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/same-slo-two-shapes.html" target="_blank" rel="noopener noreferrer">open</a> |
| 4 | Two workload classes | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/two-workload-classes.html" target="_blank" rel="noopener noreferrer">open</a> |
| 4 | Error budget | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/error-budget.html" target="_blank" rel="noopener noreferrer">open</a> |
| 4 | Choosing targets | <a href="https://amandavarella.github.io/google-sre-book/chapters/ch04-service-level-objectives/diagrams/choosing-targets.html" target="_blank" rel="noopener noreferrer">open</a> |

## Running locally

The diagrams are plain HTML — open the file directly, or serve the repo:

```sh
python3 -m http.server 8000
# then visit http://localhost:8000
```

## A note on the source material

*Site Reliability Engineering* is [free to read online](https://sre.google/sre-book/table-of-contents/).
Everything in this repo is my own notes and my own redrawn diagrams; no book text is
reproduced here.
