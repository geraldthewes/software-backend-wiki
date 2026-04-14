# CAP Theorem and PACELC

> **Tier 2** | Source: Brewer (2000), Abadi (2012) | Derives From: ka02-architecture | Authority: established practice

## Summary

The CAP Theorem defines the fundamental trade-offs in distributed data storage. PACELC extends it to account for latency even in the absence of network partitions. Together they provide the framework for choosing a data store that matches the consistency and availability requirements of a given service.

## Key Concepts

### CAP Theorem (Brewer 2000)

Eric Brewer conjectured (proved by Gilbert and Lynch in 2002) that a distributed data store can provide at most **two** of these three guarantees simultaneously:

| Guarantee | Definition |
|-----------|------------|
| **C — Consistency** | Every read receives the most recent write, or an error. All nodes see the same data at the same time. |
| **A — Availability** | Every request receives a response (not necessarily the most recent data). The system remains operational. |
| **P — Partition Tolerance** | The system continues to operate despite an arbitrary number of messages being dropped or delayed between nodes. |

**In practice, P is non-negotiable.** Network partitions happen in any real distributed system (hardware failure, network congestion, routing issues). The real choice is: **during a partition, do you prioritize C or A?**

#### CP Systems (Consistency + Partition Tolerance)

During a partition, the system refuses requests it cannot answer consistently (returns error or waits). Data is never stale.

**Best for**: financial transactions, inventory management, payments, any domain where stale reads cause incorrect business decisions.

**Examples**: PostgreSQL (strong consistency mode), HBase, Zookeeper, etcd.

#### AP Systems (Availability + Partition Tolerance)

During a partition, the system continues to serve requests but may return stale data. Nodes reconcile divergent state eventually.

**Best for**: social media feeds, DNS, shopping carts, search indexes, any domain where temporary staleness is acceptable.

**Examples**: Cassandra, DynamoDB, CouchDB, Riak.

---

### PACELC Theorem (Abadi 2012)

CAP only addresses behavior **during** a partition. PACELC extends the analysis to cover the normal (non-partition) operating case:

> **If there is a Partition (P), choose between Availability (A) and Consistency (C). Else (E), even without a partition, choose between Latency (L) and Consistency (C).**

The PACELC classification is written as two pairs: `PA/EL`, `PC/EC`, `PC/EL`, etc.

| Classification | During Partition | Normal Operation | Characteristics |
|----------------|-----------------|-----------------|-----------------|
| **PA/EL** | Prefers Availability | Prefers Low Latency | Maximum performance; eventual consistency |
| **PC/EC** | Prefers Consistency | Prefers Strong Consistency | Maximum correctness; higher latency |
| **PC/EL** | Prefers Consistency | Prefers Low Latency | Fast reads, consistent under partition |
| **PA/EC** | Prefers Availability | Prefers Strong Consistency | Available under partition, consistent normally |

---

### Data Store Decision Table

| Data Store | CAP Class | PACELC | Best Use Case |
|------------|-----------|--------|---------------|
| **PostgreSQL** | CP | PC/EC | Financial transactions, relational data, strong consistency required |
| **Cassandra** | AP | PA/EL | High-write workloads, globally distributed, eventual consistency acceptable |
| **Redis** | CP | PC/EL | Caching, session storage, leaderboards — fast reads, consistent under partition |
| **DynamoDB** | AP (default) | PA/EL | AWS-native scale-first workloads, eventual consistency |
| **MongoDB** | CP (default) | Configurable | Document storage with flexible schema; write concern tunable |
| **HBase** | CP | PC/EC | Large-scale structured data, strong consistency needed (Hadoop ecosystem) |
| **CouchDB** | AP | PA/EL | Offline-first sync, multi-master replication |
| **Zookeeper / etcd** | CP | PC/EC | Distributed coordination, leader election, configuration storage |

---

### Agent Decision Protocol

When selecting a data store, answer these questions in order:

1. **What is the consequence of a stale read?**
   - "Money is debited twice" / "Inventory goes negative" → high C priority → choose CP / PC/EC
   - "User sees a post 2 seconds late" → low C priority → AP / PA/EL is acceptable

2. **What is the consequence of downtime?**
   - "Payment cannot be processed" → medium A priority, but C still wins for correctness
   - "Search is temporarily unavailable" → high A priority → choose AP

3. **Apply PACELC for latency baseline**
   - High-throughput read path (e.g., user profile lookup) → prefer EL (low latency)
   - Audit log, financial ledger → prefer EC (strong consistency worth the latency)

4. **Cross-check the existing infrastructure**
   - If the cluster already has PostgreSQL and the use case fits, prefer reuse over adding a new data store
   - Adding Cassandra for a use case that PostgreSQL handles is operational complexity without benefit

---

### Python Example — Choosing Between Consistency Modes

PostgreSQL read committed vs. serializable:

```python
import psycopg

# Default: READ COMMITTED — may see data committed by concurrent transactions
with psycopg.connect(DATABASE_URL) as conn:
    conn.execute("BEGIN")
    result = conn.execute("SELECT balance FROM accounts WHERE id = %s", (account_id,))

# SERIALIZABLE — maximum consistency; appropriate for financial operations
with psycopg.connect(DATABASE_URL) as conn:
    conn.execute("BEGIN ISOLATION LEVEL SERIALIZABLE")
    balance = conn.execute(
        "SELECT balance FROM accounts WHERE id = %s FOR UPDATE", (account_id,)
    ).fetchone()[0]
    if balance >= amount:
        conn.execute(
            "UPDATE accounts SET balance = balance - %s WHERE id = %s",
            (amount, account_id)
        )
    conn.execute("COMMIT")
```

## Agent Guidance

### Do
- Use the decision protocol above before every data store selection
- Document the CAP/PACELC classification and reasoning in the service's architecture notes
- Choose CP/PC/EC for any data that drives financial, inventory, or access-control decisions
- Choose AP/PA/EL for high-availability read paths where eventual consistency is acceptable

### Do Not
- Do not default to the most popular data store without considering the consistency requirements
- Do not mix AP and CP expectations — if you need strong consistency, do not use an AP store
- Do not assume "consistency" means the same thing in every data store's documentation — verify the specific isolation level

## Checklist
- [ ] Consistency requirements are documented before data store selection
- [ ] Data store is classified by CAP and PACELC
- [ ] Financial, inventory, and access-control data uses a CP store
- [ ] AP stores are only used where eventual consistency is explicitly acceptable
- [ ] Isolation levels are explicitly set for critical transactions (not left at the driver default)

## See Also
- `wiki/tier2-core/distributed-systems/fallacies.md`
- `wiki/tier2-core/distributed-systems/resilience-patterns.md`
- `wiki/tier2-core/distributed-systems/overview.md`
- `wiki/tier2-core/twelve-factor-app/factors.md`
- `wiki/tier1-sources/swebok-v4/ka02-architecture.md`

## Source

Eric Brewer, "Towards Robust Distributed Systems" (PODC 2000); Gilbert & Lynch proof (2002). Daniel Abadi, "Consistency Tradeoffs in Modern Distributed Database System Design" (2012). Synthesized from *Software Development Best Practices for Agent* reference document.
