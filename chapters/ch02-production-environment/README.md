# Chapter 02 — The Production Environment at Google, from the Viewpoint of an SRE

> Part I. Introduction of *Site Reliability Engineering* (Google / O'Reilly).
>
> Notes last synced 2026-07-25.

---

## Interactive diagrams

| Diagram | Open |
| --- | --- |
| **Life of a Request**: a step-through of Figure 2-4, redrawn to put the browser in the middle and to show the response path the original leaves out | [Open](https://amandavarella.github.io/google-sre-book/chapters/ch02-production-environment/diagrams/life-of-a-request.html) |

### Diagram walkthrough

The book's Figure 2-4 walks a Shakespeare-search query through Google's stack. Two things
worth being explicit about that the original figure glosses over:

1. **The user never talks to any of these systems.** The browser does. DNS resolution, the
   TCP connection, the HTTP request — all of it is the browser acting on the user's behalf.
2. **There is a whole return trip.** The figure stops at Bigtable. In reality the reply
   protobuf climbs back up backend → frontend → GFE → browser, and only then is anything
   rendered.

**The GFE is a reverse proxy.** A forward proxy sits in front of *clients* and hides them from
the internet. A reverse proxy sits in front of *servers* and hides them from clients. The
browser's TCP connection terminates at the Google Front End; it never reaches the Shakespeare
frontend, the backend, or Bigtable. That indirection is what buys one stable address the client
has to know forever, TLS termination in one place, and free substitution of backends — GSLB picks
a healthy one per request, and the client is none the wiser when a server dies.

**GSLB shows up three times.** It's easy to read GSLB as "the load balancer" and picture one hop.
It gets consulted at three different layers within this single request:

| Who asks | What they want |
| --- | --- |
| DNS server | which frontend *IP* to hand this user, based on regional load |
| GFE | a healthy application frontend to forward the RPC to |
| App frontend | an unloaded backend to send the protobuf to |

<!-- obsidian:start -->

## Notes

### RPC

RPC (Remote Procedure Call) solves the problem of making network communication look and feel like
a local function call.

#### Life before RPC

Before RPC, if you wanted one program to ask another program (running on a different machine) to
do something, you had to:

- Manually open a network socket
- Design your own message format (what bytes mean what)
- Serialize your data into that format by hand
- Send it over the wire
- Parse the response back out on the other end
- Handle partial reads, timeouts, and connection failures yourself
- Repeat all of this, by hand, for every single operation you wanted to expose

This was tedious and error-prone. Every team invented its own wire protocol. Two systems built by
different teams often couldn't talk to each other without custom glue code. The plumbing of "send
bytes, get bytes back" dominated the code, and the actual business logic you cared about was
buried under it.

#### What RPC changed

RPC's core idea: let a programmer call a function on a remote machine the same way they'd call a
function in their own code, for example `getUser(id)`, and have the underlying system handle:

- Serializing the arguments
- Sending them over the network
- Invoking the matching function on the remote machine
- Serializing the result
- Sending it back and deserializing it locally

This is often called "location transparency": the caller doesn't need to think about whether the
function is local or remote. The mechanics of transport, encoding, and error handling are
abstracted away by the RPC framework (early examples: Sun RPC in the 1980s; modern examples: gRPC,
Thrift, JSON-RPC).

#### Trade-off worth knowing

RPC hides the network, but the network doesn't go away. A remote call can fail in ways a local
call never does (timeout, partial failure, network partition), and treating it as "just a function
call" can hide that complexity rather than solve it. This is a well-known critique of early RPC
design (see Waldo et al., *A Note on Distributed Computing*, 1994), not a settled consensus, so
worth flagging as a real design tension rather than a solved problem.

Often, an RPC call is made even when a call to a subroutine in the local program needs to be
performed. This makes it easier to refactor the call into a different server if more modularity is
needed or when a server's codebase grows. An open source version of Google's RPC is called gRPC.

A server receives RPC requests from its front end and sends RPC to its backend.

Data is transferred to and from an RPC using protocol buffers, or protobufs. Protobufs have many
advantages over XML for serialising structured data:

- Simpler to use
- 3 to 10 times smaller
- 20 to 100 times faster
- Less ambiguous

#### Protobuffs

Protocol Buffers (protobuf) is Google's language-neutral format for serializing structured data,
so you can define a data structure once and use it consistently across different programming
languages and systems.

**The core problem it solves.** When two systems need to exchange data (for example, over RPC),
you need a shared format both sides agree on. Options like JSON or XML work but are verbose as
text, slower to parse, and don't enforce a strict schema. Protobuf solves this by:

- Defining data structures in a compact, typed schema file (`.proto`)
- Compiling that schema into native code for whatever language you're using (Python, Go, Java,
  C++, and so on)
- Encoding the actual data as binary, not text, which is smaller and faster to parse

**How it works in practice.** You write a `.proto` file describing your data, something like:

```protobuf
message Person {
  string name = 1;
  int32 id = 2;
  string email = 3;
}
```

The protobuf compiler (`protoc`) then generates code in your target language with classes or
structs matching that schema, plus methods to serialize and deserialize instances of it. Every
field gets a number (the `= 1`, `= 2` above), which is how the binary format identifies fields
without needing field names in the payload, keeping it compact.

**Why it's often paired with gRPC.** gRPC (a modern RPC framework) uses protobuf by default to
define both the data structures and the service interface (which functions are callable, what they
take, what they return) in the same `.proto` file. Protobuf is often the piece that defines the
"shape" of the remote calls that RPC then handles.

**Trade-offs worth knowing.**

- Binary format means it's not human-readable like JSON, so debugging raw payloads is harder
  without tooling
- Schema changes need care: you can add fields safely, but renumbering or removing fields
  carelessly breaks compatibility with older clients
- Requires a compile step, which adds friction compared to just sending JSON, though this is
  usually worth it for the size and speed gains

#### RPC x APIs

One useful way to think about the difference between RPC and APIs: RPC tends to dominate internal,
service-to-service communication where speed and strict typing matter, while REST or GraphQL tend
to dominate public APIs where human readability and broad client compatibility matter more.

## Key takeaways

-

<!-- obsidian:end -->

## Questions / things to revisit

-
