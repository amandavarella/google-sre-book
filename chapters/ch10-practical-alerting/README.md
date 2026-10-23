# Chapter 10 — Practical Alerting from Time-Series Data

> Part III. Practices of *Site Reliability Engineering* (Google / O'Reilly).

---

## Notes

### SLOs for internal issues and incidents

From the 2nd edition. The authors' team also set SLOs (service level objectives, see
<a href="https://github.com/amandavarella/google-sre-book/blob/main/chapters/ch04-service-level-objectives/README.md#objectives" target="_blank" rel="noopener noreferrer">Chapter 4: Objectives</a>)
for handling high-priority internal issues, with reporting and clear expectations. Two of
those SLOs:

- Time to give an internal status update.
- Time to resolve the issue.

For incidents, they set and enforce goals for these metrics:

| Metric | What a high number points to |
|------|------|
| Percentage of unactionable incidents (incidents where there was nothing to do) | Monitoring that is noisy or set up badly. |
| Percentage of manually detected incidents (users or other teams reported the problem first) | SLIs (service level indicators, see <a href="https://github.com/amandavarella/google-sre-book/blob/main/chapters/ch04-service-level-objectives/README.md#indicators" target="_blank" rel="noopener noreferrer">Chapter 4: Indicators</a>) that are defined wrong, or monitoring that does not cover enough. |
| Pages per incident (pages: on-call alerts) | Too many alerts for one incident. They can distract the on-call engineer and make it harder to work out what is happening. |

## Key takeaways

- (2nd edition) Incident handling gets its own SLOs and goals: time to status update, time to
  resolve, and the rates of unactionable incidents, manually detected incidents, and pages per
  incident.

## Interactive diagrams

_None yet._

## Questions / things to revisit

- 
