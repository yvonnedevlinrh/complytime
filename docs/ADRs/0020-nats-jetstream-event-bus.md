# ADR-0020: NATS JetStream as Event Bus

| Field | Value |
|:------|:------|
| Status | accepted |
| Date | 2026-07-29 |
| Decision-makers | @jpower432 |

## Context and Problem Statement

[ADR-0019](0019-event-driven-ingestion.md) established that the ComplyTime API uses a pub/sub event bus to notify consumers of ingested evidence. Which event bus technology should be used? The bus must provide durable message delivery, consumer acknowledgment, dead-letter handling, and broker-side filtering without imposing significant operational overhead.

## Decision Drivers

* Durable delivery — messages must survive broker restarts and consumer downtime.
* At-least-once semantics with consumer acknowledgment and redelivery.
* Dead-letter support for undeliverable messages.
* Broker-side subject filtering so consumers receive only the event types they need.
* Operational simplicity — no dedicated messaging infrastructure team; contributors must be able to operate the bus without specialist knowledge.
* Cloud-native deployment model (Kubernetes).
* License compatibility with Apache 2.0.

## Considered Options

* NATS with JetStream
* Apache Kafka
* RabbitMQ

## Decision Outcome

Chosen option: "NATS with JetStream", because it provides durable, acknowledged delivery with dead-letter support in a lightweight, Kubernetes-native package that matches the operational scale of the project, under an Apache 2.0 license.

### Confirmation

Ingestion events are published to a JetStream stream. A test consumer with `max_deliver` configured receives events, fails to acknowledge them, and confirms messages land in a dead-letter stream after exhausting retries. Broker restart does not lose in-flight messages.

## Consequences

* Good, because JetStream adds durable streams and consumer acks on top of NATS core — no separate persistence layer required.
* Good, because NATS deploys as a single binary or small cluster with a Kubernetes operator available.
* Good, because hierarchical subject naming (`domain.action.entity`) enables broker-side filtering without consumer-side logic.
* Good, because NATS server and Go client are Apache 2.0, matching this project's license.
* Bad, because plain NATS (without JetStream) is fire-and-forget; JetStream must be explicitly enabled and configured to avoid silent message loss.
* Bad, because JetStream stream and consumer configuration (retention policy, max deliver, DLQ stream) must be defined and maintained as infrastructure code.

## Pros and Cons of the Options

### NATS with JetStream

NATS core is a lightweight pub/sub broker. JetStream adds persistent streams, consumer acknowledgment, replay, and dead-letter streams as a first-class feature. Licensed Apache 2.0.

* Good, because operational footprint requires no specialist knowledge — single binary, minimal tuning required.
* Good, because JetStream consumer model maps directly to at-least-once delivery with per-consumer state tracked by the broker.
* Good, because subject hierarchy and wildcard subscriptions (`>`) enable broker-side filtering without additional infrastructure.
* Bad, because JetStream must be explicitly enabled; teams new to NATS may default to core NATS and miss durability guarantees.

### Apache Kafka

Kafka is a distributed commit log designed for high-throughput, ordered event streaming at scale. Licensed Apache 2.0.

* Good, because battle-tested at very high throughput with a rich ecosystem of connectors and tooling.
* Neutral, because the operational model (brokers, partitions, consumer groups, offset management) is well-understood but sized for scale beyond compliance evidence ingestion volume.
* Bad, because cluster management and tuning overhead exceeds what a smaller teams can sustain without dedicated infrastructure support.
* Bad, because Kafka's topic-per-event-type model shifts broker-side filtering responsibility to the consumer or requires additional tooling (Kafka Streams, ksqlDB), adding operational surface area.

### RabbitMQ

RabbitMQ is a mature AMQP broker with strong delivery guarantees, dead letter exchanges, and a well-documented operational model. Licensed MPL-2.0.

* Good, because dead letter exchanges are a first-class primitive with flexible routing.
* Good, because quorum queues provide strong durability guarantees.
* Neutral, because the AMQP exchange/queue/binding model is more expressive than needed for this use case.
* Neutral, because a Kubernetes operator exists but RabbitMQ's stateful cluster management (node identity, disk persistence, peer discovery) adds operational complexity relative to NATS.
* Neutral, because MPL-2.0 is compatible with Apache 2.0 projects when used as an external service, but introduces a license boundary worth tracking.
* Bad, because protocol and configuration complexity is higher than NATS for teams building their own consumers from scratch.
* Bad, because exchange/queue routing does not provide subject-hierarchy filtering natively; consumer-side filtering or per-queue routing rules are required for event-type fanout.

## More Information

* [ADR-0019: Event-Driven Ingestion Pattern](0019-event-driven-ingestion.md)
* [NATS JetStream documentation](https://docs.nats.io/nats-concepts/jetstream)
* [NATS license (Apache 2.0)](https://github.com/nats-io/nats-server/blob/main/LICENSE)
* [RabbitMQ license (MPL-2.0)](https://github.com/rabbitmq/rabbitmq-server/blob/main/LICENSE)
