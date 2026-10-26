# 05 — Phase 4: Distributed Systems, Consistency and Availability

> **Week 4** · Learn the trade-offs that matter at scale.

## Checklist

**1. CAP theorem and consistency**

- [ ] CAP theorem
- [ ] Network partitions
- [ ] CP vs. AP systems
- [ ] Strong consistency
- [ ] Eventual consistency
- [ ] Weak consistency
- [ ] Read-after-write consistency
- [ ] Quorum reads and writes
- [ ] Consistency vs. availability trade-offs

**2. Availability patterns**

- [ ] Active-active architecture
- [ ] Active-passive architecture
- [ ] Failover and automatic recovery
- [ ] Health checks and failure detection
- [ ] Replication
- [ ] Leader election
- [ ] Split-brain scenarios

**3. Replication patterns**

- [ ] Primary-replica (formerly master-slave)
- [ ] Multi-primary (formerly master-master)
- [ ] Synchronous vs. asynchronous replication
- [ ] Replication lag
- [ ] Read replicas
- [ ] Conflict resolution

**4. Availability mathematics**

- [ ] 99.9% vs. 99.99% vs. 99.999% availability
- [ ] Downtime budgets
- [ ] Availability in parallel vs. sequential dependencies
- [ ] Failure domains
- [ ] Recovery Time Objective (RTO)
- [ ] Recovery Point Objective (RPO)

## 1. CAP theorem and consistency

> **Remember:** CAP concerns consistency and availability when a network partition occurs. It doesn't mean every system simply chooses two properties under all circumstances.

### CAP theorem

- **Notes:**

### Network partitions

- **Notes:**

### CP vs. AP systems

- **Notes:**

### Strong consistency

- **Notes:**

### Eventual consistency

- **Notes:**

### Weak consistency

- **Notes:**

### Read-after-write consistency

- **Notes:**

### Quorum reads and writes

- **Notes:**

### Consistency vs. availability trade-offs

- **Notes:**

## 2. Availability patterns

### Active-active architecture

- **Notes:**

### Active-passive architecture

- **Notes:**

### Failover and automatic recovery

- **Notes:**

### Health checks and failure detection

- **Notes:**

### Replication

- **Notes:**

### Leader election

- **Notes:**

### Split-brain scenarios

- **Notes:**

## 3. Replication patterns

### Primary-replica (formerly master-slave)

- **Notes:**

### Multi-primary (formerly master-master)

- **Notes:**

### Synchronous vs. asynchronous replication

- **Notes:**

### Replication lag

- **Notes:**

### Read replicas

- **Notes:**

### Conflict resolution

- **Notes:**

## 4. Availability mathematics

| Availability | Approx. downtime per year |
| --- | --- |
| 99% | 3 days, 15 hours |
| 99.9% | 8 hours, 46 minutes |
| 99.99% | 52 minutes, 34 seconds |
| 99.999% | 5 minutes, 15 seconds |

### 99.9% vs. 99.99% vs. 99.999% availability

- **Notes:**

### Downtime budgets

- **Notes:**

### Availability in parallel vs. sequential dependencies

- **Notes:**

### Failure domains

- **Notes:**

### Recovery Time Objective (RTO)

- **Notes:**

### Recovery Point Objective (RPO)

- **Notes:**

## Practice

**Exercise:** Design a highly available authentication service and explain what happens if one database replica or an entire availability zone fails.

- **Requirements:**
- **Scale estimation:**
- **APIs:**
- **Data model:**
- **High-level architecture:**
- **Bottlenecks & deep dive:**
- **Failure handling:**
- **Trade-offs:**
