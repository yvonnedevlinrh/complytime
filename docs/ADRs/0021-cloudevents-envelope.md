# ADR-0021: CloudEvents for Ingestion Event Envelope

| Field | Value |
|:------|:------|
| Status | accepted |
| Date | 2026-07-29 |
| Decision-makers | @jpower432 |

## Context and Problem Statement

[ADR-0019](0019-event-driven-ingestion.md) established that the ComplyTime API publishes ingestion events to a pub/sub event bus. Consumers need a stable, language-neutral envelope format to parse events without coupling to a proprietary schema. What format should ingestion events use?

## Decision Drivers

* Language-neutral — consumers written in any language can parse events without a ComplyTime-specific SDK.
* Stable, versioned specification so consumers can rely on envelope structure across ComplyTime API releases.
* Broad tooling support for routing, filtering, and validation.
* License compatible with Apache 2.0.

## Considered Options

* CloudEvents (CNCF specification)
* Custom JSON envelope
* Protobuf-defined envelope

## Decision Outcome

Chosen option: "CloudEvents", because it is a CNCF-standardized, language-neutral envelope with broad ecosystem tooling and a stable specification, reducing the burden on consumers to understand a proprietary format.

### Confirmation

Ingestion events published to the NATS JetStream stream conform to the CloudEvents specification. A consumer using the CloudEvents SDK in any supported language can parse the envelope without ComplyTime-specific parsing logic.

## Consequences

* Good, because consumers can use CloudEvents SDKs (available in Go, Java, Python, JavaScript, and others) rather than hand-rolling envelope parsing.
* Good, because CloudEvents is transport-agnostic — the same envelope works over NATS, HTTP, or any future transport.
* Good, because the `type` and `source` attributes enable broker-side filtering without inspecting event payload.
* Good, because CloudEvents specification is Apache 2.0 licensed.
* Bad, because the CloudEvents envelope adds required attributes (`id`, `source`, `specversion`, `type`) that must be populated correctly on every published event.

## Pros and Cons of the Options

### CloudEvents (CNCF specification)

A vendor-neutral specification for describing event data in a common way. Maintained by the CNCF with SDKs across major languages. Licensed Apache 2.0.

* Good, because the specification is externally maintained and documented — consumers reference [cloudevents.io](https://cloudevents.io) rather than ComplyTime docs for envelope semantics.
* Good, because the CloudEvents NATS Protocol Binding defines how to carry CloudEvents over NATS; envelope conformance is enforced at the application layer, not the broker.
* Bad, because adopting the specification means conforming to its required attributes even for simple internal events.

### Custom JSON envelope

A ComplyTime-defined JSON structure for event metadata.

* Good, because full control over the schema and evolution.
* Bad, because consumers must understand a proprietary format with no external documentation.
* Bad, because tooling (routing, filtering, validation) must be built or adapted rather than reused from the CloudEvents ecosystem.

### Protobuf-defined envelope

A binary envelope defined in Protocol Buffers, with generated clients per language.

* Good, because compact binary encoding and strong typing.
* Good, because schema evolution rules are well-defined.
* Bad, because consumers must generate or import Protobuf bindings, adding a build-time dependency on ComplyTime's proto definitions.
* Bad, because binary encoding is harder to inspect during development and debugging.

## More Information

* [ADR-0019: Event-Driven Ingestion Pattern](0019-event-driven-ingestion.md)
* [ADR-0020: NATS JetStream as Event Bus](0020-nats-jetstream-event-bus.md)
* [CloudEvents specification](https://cloudevents.io)
* [CloudEvents GitHub (Apache 2.0)](https://github.com/cloudevents/spec)
