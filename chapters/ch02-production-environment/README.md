# Chapter 02 — The Production Environment at Google, from the Viewpoint of an SRE

> Part I. Introduction of *Site Reliability Engineering* (Google / O'Reilly).

---

## Interactive diagrams

| Diagram | Open |
| --- | --- |
| **Life of a Request** — a step-through of Figure 2-4, redrawn to put the browser in the middle and to show the response path the original leaves out | [View live](https://amandavarella.github.io/google-sre-book/chapters/ch02-production-environment/diagrams/life-of-a-request.html) · [source](diagrams/life-of-a-request.html) |

## Notes

### Life of a request

The book's Figure 2-4 walks a Shakespeare-search query through Google's stack. Two things
worth being explicit about that the original figure glosses over:

1. **The user never talks to any of these systems.** The browser does. DNS resolution, the
   TCP connection, the HTTP request — all of it is the browser acting on the user's behalf.
2. **There is a whole return trip.** The figure stops at Bigtable. In reality the reply
   protobuf climbs back up backend → frontend → GFE → browser, and only then is anything
   rendered.

### The GFE is a reverse proxy

A forward proxy sits in front of *clients* and hides them from the internet. A **reverse
proxy** sits in front of *servers* and hides them from clients.

The browser's TCP connection terminates at the Google Front End. It never reaches the
Shakespeare frontend, the backend, or Bigtable. That indirection is what buys:

- one stable address the client has to know, forever
- TLS termination in one place
- free substitution of backends — GSLB picks a healthy one per request, and the client is
  none the wiser when a server dies

### GSLB shows up three times

It's easy to read GSLB as "the load balancer" and picture one hop. It gets consulted at
three different layers within this single request:

| Who asks | What they want |
| --- | --- |
| DNS server | which frontend *IP* to hand this user, based on regional load |
| GFE | a healthy application frontend to forward the RPC to |
| App frontend | an unloaded backend to send the protobuf to |

## Key takeaways

-

## Questions / things to revisit

-
