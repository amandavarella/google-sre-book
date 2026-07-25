# Google SRE Book — Learning Notes

My notes, takeaways and **interactive diagrams** while working through
[*Site Reliability Engineering*](https://sre.google/books/) (Beyer, Jones, Petoff & Murphy — O'Reilly, 2016).

📖 **[Browse the interactive version →](https://amandavarella.github.io/google-sre-book/)**

## How this repo is organised

```
README.md                         you are here
index.html                        GitHub Pages landing page
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
| 1 | Introduction | [notes](chapters/ch01-introduction/README.md) | — |
| 2 | The Production Environment at Google, from the Viewpoint of an SRE | [notes](chapters/ch02-production-environment/README.md) | 1 |
| 3 | Embracing Risk | [notes](chapters/ch03-embracing-risk/README.md) | — |
| 4 | Service Level Objectives | [notes](chapters/ch04-service-level-objectives/README.md) | — |
| 5 | Eliminating Toil | [notes](chapters/ch05-eliminating-toil/README.md) | — |
| 6 | Monitoring Distributed Systems | [notes](chapters/ch06-monitoring-distributed-systems/README.md) | — |
| 7 | The Evolution of Automation at Google | [notes](chapters/ch07-evolution-of-automation/README.md) | — |
| 8 | Release Engineering | [notes](chapters/ch08-release-engineering/README.md) | — |
| 9 | Simplicity | [notes](chapters/ch09-simplicity/README.md) | — |
| 10 | Practical Alerting from Time-Series Data | [notes](chapters/ch10-practical-alerting/README.md) | — |
| 11 | Being On-Call | [notes](chapters/ch11-being-on-call/README.md) | — |
| 12 | Effective Troubleshooting | [notes](chapters/ch12-effective-troubleshooting/README.md) | — |
| 13 | Emergency Response | [notes](chapters/ch13-emergency-response/README.md) | — |
| 14 | Managing Incidents | [notes](chapters/ch14-managing-incidents/README.md) | — |
| 15 | Postmortem Culture: Learning from Failure | [notes](chapters/ch15-postmortem-culture/README.md) | — |
| 16 | Tracking Outages | [notes](chapters/ch16-tracking-outages/README.md) | — |
| 17 | Testing for Reliability | [notes](chapters/ch17-testing-for-reliability/README.md) | — |
| 18 | Software Engineering in SRE | [notes](chapters/ch18-software-engineering-in-sre/README.md) | — |
| 19 | Load Balancing at the Frontend | [notes](chapters/ch19-load-balancing-frontend/README.md) | — |
| 20 | Load Balancing in the Datacenter | [notes](chapters/ch20-load-balancing-datacenter/README.md) | — |
| 21 | Handling Overload | [notes](chapters/ch21-handling-overload/README.md) | — |
| 22 | Addressing Cascading Failures | [notes](chapters/ch22-cascading-failures/README.md) | — |
| 23 | Managing Critical State: Distributed Consensus for Reliability | [notes](chapters/ch23-distributed-consensus/README.md) | — |
| 24 | Distributed Periodic Scheduling with Cron | [notes](chapters/ch24-distributed-cron/README.md) | — |
| 25 | Data Processing Pipelines | [notes](chapters/ch25-data-processing-pipelines/README.md) | — |
| 26 | Data Integrity: What You Read Is What You Write | [notes](chapters/ch26-data-integrity/README.md) | — |
| 27 | Reliable Product Launches at Scale | [notes](chapters/ch27-reliable-product-launches/README.md) | — |
| 28 | Accelerating SREs to On-Call and Beyond | [notes](chapters/ch28-accelerating-sres/README.md) | — |
| 29 | Dealing with Interrupts | [notes](chapters/ch29-dealing-with-interrupts/README.md) | — |
| 30 | Embedding an SRE to Recover from Operational Overload | [notes](chapters/ch30-embedding-an-sre/README.md) | — |
| 31 | Communication and Collaboration in SRE | [notes](chapters/ch31-communication-collaboration/README.md) | — |
| 32 | The Evolving SRE Engagement Model | [notes](chapters/ch32-sre-engagement-model/README.md) | — |
| 33 | Lessons Learned from Other Industries | [notes](chapters/ch33-lessons-from-other-industries/README.md) | — |
| 34 | Conclusion | [notes](chapters/ch34-conclusion/README.md) | — |

Parts follow the book: **I. Introduction** (1–2) · **II. Principles** (3–9) ·
**III. Practices** (10–27) · **IV. Management** (28–32) · **V. Conclusions** (33–34).

## Interactive diagrams

| Chapter | Diagram | View |
| --- | --- | --- |
| 2 | Life of a Request | [open](https://amandavarella.github.io/google-sre-book/chapters/ch02-production-environment/diagrams/life-of-a-request.html) |

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
