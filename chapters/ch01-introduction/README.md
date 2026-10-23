# Chapter 01 — Introduction

> Part I. Introduction of *Site Reliability Engineering* (Google / O'Reilly).
>
> Notes last synced 2026-07-25.

---

<!-- obsidian:start -->

## Notes

### Provisioning

Provisioning combines both change management and capacity planning. It must be conducted quickly
and only when necessary, because capacity is expensive, and it must be done correctly or the
capacity won't work when needed.

- Adding new capacity often means spinning up a new instance or location, making significant
  modifications to existing systems (configuration files, load balancers, networking), and
  validating that the new capacity performs and delivers correct results.
- It is a riskier operation than load shifting, which is often done multiple times per hour, so
  it must be treated with a corresponding degree of extra caution.

### Emergency Response

Reliability is a function of mean time to failure (MTTF) and mean time to repair (MTTR). The most
relevant metric for evaluating emergency response is how quickly the response team can bring the
system back to health, that is, the MTTR.

- Humans add latency. A system that avoids emergencies requiring human intervention will have
  higher availability than one that requires hands-on intervention, even if it experiences more
  actual failures.
- Thinking through and recording best practices ahead of time in a "playbook" produces roughly a
  3x improvement in MTTR compared to "winging it".
- No playbook, however comprehensive, substitutes for smart engineers who can think on the fly,
  but clear, thorough troubleshooting steps and tips are valuable when responding to a
  high-stakes or time-sensitive page.
- Google SRE relies on on-call playbooks, plus exercises such as the "Wheel of Misfortune", to
  prepare engineers to react to on-call events.

### Error budget

If 100% is the wrong reliability target for a system, the right target is a product question, not
a technical one. It should take into account:

- What level of availability will users be happy with, given how they use the product?
- What alternatives are available to users who are dissatisfied with the product's availability?
- What happens to users' usage of the product at different availability levels?

The business or product must establish the availability target. Once set, the error budget is one
minus the availability target: a service that is 99.99% available is 0.01% unavailable, and that
permitted 0.01% is the error budget, which can be spent on anything as long as it isn't overspent.

- **How to spend it:** the development team wants to launch features and attract users. Ideally
  the whole budget is spent taking launch risks to ship quickly. Freeing up the error budget
  through tactics such as phased rollouts and 1% experiments optimizes for quicker launches.
- The error budget resolves the structural conflict of incentives between development and SRE.
  The goal is no longer "zero outages"; instead both aim to spend the budget for maximum feature
  velocity.
- An outage is no longer a "bad" thing. It is an expected part of the process of innovation, and
  an occurrence that both development and SRE teams manage rather than fear.

### Efficiency and Performance

Efficient use of resources matters any time a service cares about money. Because SRE ultimately
controls provisioning, it must also be involved in any work on utilization, since utilization is a
function of how a service works and how it is provisioned. Paying close attention to provisioning
strategy, and therefore utilization, is a very big lever on a service's total costs.

- Resource use is a function of demand (load), capacity, and software efficiency. SREs predict
  demand, provision capacity, and can modify the software. These three factors are a large part
  (though not the entirety) of a service's efficiency.
- Software systems become slower as load is added to them; a slowdown equates to a loss of
  capacity. At some point a slowing system stops serving, which corresponds to infinite slowness.
- SREs provision to meet a capacity target at a specific response speed, so they are keenly
  interested in performance. SREs and product developers monitor and modify a service to improve
  its performance, thus adding capacity and improving efficiency.

### Demand Forecasting and Capacity Planning

Demand forecasting and capacity planning ensure there is sufficient capacity and redundancy to
serve projected future demand with the required availability. The concepts aren't special, except
that a surprising number of services and teams don't take the steps needed to have the required
capacity in place by the time it is needed. Planning must account for both organic growth (natural
product adoption and usage by customers) and inorganic growth (feature launches, marketing
campaigns, or other business-driven changes).

Several steps are mandatory in capacity planning:

- An accurate organic demand forecast, which extends beyond the lead time required for acquiring
  capacity.
- An accurate incorporation of inorganic demand sources into the demand forecast.
- Regular load testing of the system to correlate raw capacity (servers, disks, and so on) to
  service capacity.

Because capacity is critical to availability, the SRE team must be in charge of capacity planning,
which means they must also be in charge of provisioning.

### A modern hierarchy of production health

From the 2nd edition (Figure 1-2). A pyramid of six layers, listed here top to bottom as drawn.

| Layer (top to bottom) | What it covers |
|------|------|
| 6. Emergent reliability | Reliable by default |
| 5. Resilient products | Design and implementation |
| 4. Infrastructure design | CI/CD (continuous integration and delivery), testing, data management, capacity, and similar |
| 3. Continuous preparedness | Incident management, postmortems, training |
| 2. Situational awareness | Monitoring, observability, alerting, toil tracking, goal setting |
| 1. Sociotechnical foundations | A safe and empowering culture of continuous improvement |

### The Trend Toward Intent-Based Coordination Systems

From the 2nd edition. This is about the next few years after the book was published: early
moves, not settled practice.

**The shift.** Today the platform is a collection of "vending machine" services: the engineer
picks each implementation detail (a scaling policy, a database type). The industry is starting
to try **intent-based** patterns instead: the engineer declares the desired state, and the
platform works out how to get there.

- **What the engineer declares:** broad goals, for example a latency target for one region, or
  a strict budget limit.
- **What the platform does:** it reconciles those goals with the infrastructure it has
  (reconcile: keeps adjusting until the real state matches the declared goals). Its tools
  include tier selection, resource isolation, workload migration, and performance throttling.

Three emerging areas are shaping this shift:

- **Adaptive signal filtering.** Modern telemetry (the metrics, logs, and traces a system
  emits) produces too much noise. The move is away from static dashboards toward tools that
  rank information by the current situation, such as context-aware debuggers and analysis
  tools. The goal: show a human only the signals needed for the decision in front of them, so
  diagnosis is faster.
- **Embedded regulatory logic.** Compliance rules and data residency rules (where data is
  allowed to be stored) are expected to be built into the deployment workflow. Legal and
  geographic limits become part of the system's declared intent, not an afterthought.
- **Resource and sustainability constraints.** Carbon intensity, energy cost, and energy
  availability are treated as primary engineering constraints. Platforms might help move
  workloads based on where energy is available or where regional power costs change.

**What this may mean for the SRE role.** The SRE job may move from building golden paths to
defining the constraints and objective functions (the goals the platform optimizes for) that
guide the platform's decisions. The book calls this high-level policy governance.

- Success is then measured less by the uptime of individual components.
- It is measured more by whether the platform keeps a stable balance between performance, cost,
  and compliance while conditions around the world keep changing.

## Key takeaways

- SRE responsibilities interlock: emergency response, error budgets, efficiency, capacity
  planning, and provisioning all feed into reliability.
- Reliability is a function of MTTF and MTTR. Reduce MTTR with playbooks and practice (the "Wheel
  of Misfortune"), and prefer automation that keeps humans out of the loop.
- 100% is the wrong reliability target. Choose it as a product decision, then use the error budget
  (one minus the target) to balance feature velocity against stability.
- Error budgets align development and SRE incentives: an outage becomes an expected, managed cost
  of innovation rather than a failure to eliminate.
- Provisioning is riskier than load shifting, so do it quickly, only when needed, and validate
  carefully.
- Efficiency is a first-class SRE concern: utilization is a huge cost lever, and performance
  (response speed under load) is part of capacity.
- Capacity planning must cover both organic and inorganic growth, forecast beyond the
  capacity-acquisition lead time, and be validated with regular load testing.
- (2nd edition) Platforms are starting to move toward intent-based coordination: engineers
  declare goals (latency, budget, compliance, energy), and the platform reconciles them. The SRE
  role may shift toward setting those constraints and goals (policy governance).

<!-- obsidian:end -->

## Interactive diagrams

_None yet._

## Questions / things to revisit

-
