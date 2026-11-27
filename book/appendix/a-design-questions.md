# Appendix A — Design questions to explore

> [Back to index](../../README.md)

This appendix collects design questions raised in the chapters that are worth exploring on your
own. Each one gives the documented facts and pointers to sources, followed by one possible
analysis. Try to form your own answer before reading the analysis.

## A.1 Why did SONiC choose Redis?

*Raised in [Chapter 1, section 1.3.4 — Pillar 3: Shared state (Redis)](../part1/ch01-disaggregation.md#134-pillar-3--shared-state-redis).*

**Questions to think about first:**

1. What did SONiC need from its shared-state layer in 2015?
2. Which alternatives existed (RPC, message buses, SQL databases, OVSDB, a custom database), and
   what would each have given or cost?
3. Where has Redis turned out to be a limit, and how did the community respond?

### Documented facts

There is no public Microsoft document comparing databases or explaining why Redis
was chosen over alternatives. The official rationale is one sentence in the
[SONiC Architecture wiki](https://github.com/sonic-net/SONiC/wiki/Architecture): SONiC relies on
a Redis key-value database to provide *"a language independent interface"* and *"data
persistence, replication and multi-process communication"* among all SONiC subsystems. The wiki
also highlights publish/subscribe: each application subscribes only to the data it needs.

### Documented — the limits found later

The community hit Redis's limits at scale and wrote design documents about them — the best real-world evidence of the trade-offs:

- **One Redis instance became a bottleneck.** The
  [multiple database instances design](https://github.com/sonic-net/SONiC/blob/master/doc/database/multi_database_instances.md)
  notes that the single instance gets very busy during large bursts of writes, such as many
  routes created at once. The fix was to allow several Redis instances, so heavy databases don't
  slow down critical ones like APPL_DB and ASIC_DB.
- **Redis round trips were reduced with batching.** Redis pipelining in sairedis and the bulk
  APIs in the
  [201908 L3 performance and scaling work](https://github.com/Azure/SONiC/blob/89abd4938d792215b75d801e87b47ccf2c22f111/doc/L3_performance_and_scaling_enchancements_HLD.md)
  exist because sending one Redis message per route was too costly.
- **ZeroMQ for the route path.** A 2024 design proposed using **ZeroMQ (ZMQ)** between fpmsyncd
  and orchagent for routes, based on Alibaba's *BGP Loading Optimization for SONiC* analysis, to
  load very large route tables faster. In effect, the community compared Redis with ZMQ — but
  only for the most performance-critical path, years after the original design.

### One possible analysis — why Redis fit the 2015 requirements

| Requirement | What Redis offers |
|---|---|
| Very fast reads and writes (route updates) | Data held in memory — no disk I/O on each operation |
| Notify consumers when something changes | Built-in publish/subscribe and keyspace notifications |
| Simple "table → key → fields" data | Hash type fits naturally (e.g. `ROUTE_TABLE:10.0.0.0/24` → `nexthop`, `ifname`) |
| Usable from C++ and Python | Mature client libraries in every language |
| Inspect and debug easily | `redis-cli` reads and watches everything live |
| Survive a process restart | State lives outside the process that crashed |
| Small and proven | One lightweight server, widely used in cloud systems |

### One possible analysis — alternatives and why they would likely lose

| Alternative | Strength | Why it fit worse |
|---|---|---|
| Direct calls between services (RPC, e.g. gRPC/Thrift) | Fast, clear contracts | Every service must know the others; if one restarts, calls fail and state is lost. No central view of the switch |
| Message bus only (ZeroMQ, D-Bus, Kafka) | Fast event delivery | Delivers messages but doesn't keep current state — a restarted service can't ask "what is true now?" Kafka is also far too heavy for a switch |
| Embedded SQL database (SQLite) | Rich queries, transactions | Disk-oriented, no built-in change notifications, harder to share across many processes |
| OVSDB (Open vSwitch database) | Built for network state, with transactions and change monitoring | A real contemporary choice — HPE's OpenSwitch NOS (2015) used it — but more complex, schema-heavy and less widely known than Redis |
| A custom state database (like Arista's Sysdb) | Tailored exactly to the NOS | Takes years to build and is owned by one company — the opposite of SONiC's open, reuse-existing-tools approach |

Redis was the simplest widely known tool that provided **both** a store of current state **and**
change notifications. Most alternatives provide only one of the two.
