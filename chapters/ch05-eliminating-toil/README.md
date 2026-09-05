# Chapter 05 — Eliminating Toil

> Part II. Principles of *Site Reliability Engineering* (Google / O'Reilly).
>
> Notes last synced 2026-09-05.

---

## Toil Defined

**Toil** is work tied to running a production service. It is not "work I dislike." It tends to
match some of the six traits below.

You do not need all six. The more closely work matches one or more of these, the more likely
it is toil.

### Manual

- A person still has to do the steps.
- Running a script counts. The script may do the work, but the minutes a human spends
  starting it are still toil.
- Count **hands-on time** (the human at the keyboard), not **elapsed time** (how long the
  script runs).

### Repetitive

- First time, or even second time: not toil.
- Over and over: toil.
- A new problem, or a new solution: not toil.

### Automatable

- If a machine could do it as well as a human, it is toil.
- If the need for the task could be designed away, it is toil.
- If human judgment is essential, it is often not toil.
- Trap: a poorly designed service that alerts several times a day, each alert needing a
  complex human response. That judgment still counts as toil until the service is rebuilt so
  the alerts stop. (A **page** is an on-call alert.)

### Tactical

- **Tactical** means interrupt-driven and reactive: something broke, handle it now.
- The opposite is strategy-driven and proactive: a planned project.
- Handling pager alerts is toil. This type of work may never go to zero. The book says to
  keep working toward minimizing it.

### No enduring value

- After you finish, the service is in the same state as before: probably toil.
- After you finish, the service is permanently better: probably not toil, even if the work
  was **grungy** (messy, unpleasant), such as digging into old code and config and
  straightening them out.

### O(n) with service growth

- **O(n)** means the work grows in step with *n*, the size of the service. If service size,
  traffic, or user count grows, this task grows with it. That is probably toil.
- An ideally managed service can grow by **one order of magnitude** (about 10×) with no extra
  human work, except one-time work to add resources.

## Why Less Toil Is Better

The advertised goal: keep **operational work** (toil) **below 50%** of each SRE's time.

Feature development typically aims at **reliability**, **performance**, or **utilization**
(how fully you use the resources you already have). Cutting toil is often a **second-order
effect**: you aimed at the feature; less toil followed.

### Calculating Toil

Quarterly surveys of Google SREs: the **average** is about **33%** toil. That is better than
the overall target of 50%.

## What Qualifies as Engineering?

### Typical SRE activities fall into the following approximate categories

**Software engineering**

- Writing or changing **code**, plus the design and docs that go with it.
- Examples: automation scripts, tools or **frameworks** (shared toolboxes other code uses),
  service features for scale and reliability, or changes to **infrastructure code** (the
  platform under a product, not a product feature).

**Systems engineering**

- Configuring production, changing config, or documenting systems as a **one-time** effort
  that leaves a lasting improvement.
- Examples: monitoring setup and updates, load balancing configuration, server
  configuration, tuning of **OS parameters** (settings on the operating system), load
  balancer setup.
- Also: consulting on architecture, design, and **productionization** (turning a prototype
  into something that can run in production).

**Toil**

- Work tied to running a service that is repetitive, manual, and the rest of the six traits.

**Overhead**

- Admin work **not** tied to running a service. It is not toil. It is not engineering.
- Examples: hiring, HR paperwork, team or company meetings, **bug queue hygiene** (cleaning
  the ticket list), **snippets** (a short weekly writeup of what you worked on), peer reviews
  and self-assessments, training courses.

Every SRE spends at least **50%** of their time on engineering (software engineering and
systems engineering), averaged over a few quarters or a year.

- Toil is **spiky** (it comes in bursts). A steady 50% engineering every quarter may not be
  realistic. Some quarters may dip below.
- If project time averages **significantly below 50%** over the long haul, the team steps
  back and figures out what is wrong.

## Is Toil Always Bad?

A small amount of toil is not a problem: some of it is unavoidable in SRE (and in almost any
engineering role), and predictable, repetitive tasks can feel calming. It becomes a problem
in large quantities. If you are burdened with too much toil, complain loudly.

**For you**

- **Career stagnation:** too little time on projects, career progress slows or stops. Google
  rewards **grungy** work when it is inevitable and has a big positive impact. You cannot
  make a career of only that.
- **Low morale:** people have different toil limits. Everyone has a limit. Too much toil
  leads to burnout, boredom, and discontent.

**For the SRE organization** (too much toil, not enough engineering)

- **Creates confusion:** SRE presents itself as an engineering organization. People or teams
  with too much toil confuse others about that role.
- **Slows progress:** the team is less productive. **Feature velocity** (how fast new
  features ship) drops if SREs are busy with manual work and **firefighting** (handling
  breaks as they happen).
- **Sets precedent:** if you take on toil too willingly, Dev counterparts have a reason to
  send more, including operational work Devs should do. Other teams may start expecting the
  same.
- **Promotes attrition** (people leaving): you might not mind the toil. Current or future
  teammates might. Too much toil in the team's procedures pushes the best engineers to look
  for a better job.
- **Causes breach of faith:** new hires and transfers who joined with the promise of project
  work feel cheated. Morale drops.

## Key takeaways

- Toil is production work scored on six traits. You do not need all six. First time is not
  toil. Running a script still counts (hands-on time, not elapsed). After you finish: same
  state = probably toil; permanently better = probably not.
- Keep toil below 50% of each SRE's time. Google's survey average is about 33%. Feature work
  (reliability, performance, utilization) often cuts toil as a second-order effect.
- Four buckets: software engineering, systems engineering, toil, overhead. Overhead is not
  toil and not engineering. Engineering is at least 50%, averaged over a few quarters or a
  year. Some quarters can dip. A long-haul average significantly below 50% means the team
  figures out what is wrong.
- Small doses of toil can be fine. Large quantities hurt you (career, morale) and the org
  (confused role, slower features, more toil sent to SRE, people leaving, broken project-work
  promise). The book says to complain loudly.

## Interactive diagrams

_None yet._

## Questions / things to revisit

- 
