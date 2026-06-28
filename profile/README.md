# Rift

> **Rift Realtime Protocol (Rift/1)** — a modern, real-time bidirectional messaging protocol built on WebSocket, designed to replace raw `JSON.stringify`-over-socket patterns with typed frames, explicit delivery semantics, and production-grade session management.

## Why Rift?

| Feature | Socket.IO | Rift/1 |
|---------|-----------|--------|
| Framing | Custom packet protocol | Typed frame envelope (`C`/`D`/`A`/`F`/`E`) |
| Encoding | JSON only | CBOR + JSON negotiated at handshake |
| Delivery guarantees | Fire-and-forget | 6 delivery modes (AtMostOnce → DurableOrdered) |
| Session resume | Basic sid cookie | Epoch-based resume with per-topic offset tracking |
| Backpressure | None | 6 strategies + token-bucket rate limiting |
| Message types | Event only | 8 classes (Event, Command, Reply, State, Datagram, Stream, Snapshot, System) |
| Topics | String namespace | Profiles with retention, ordering, subscriber/publisher limits |

## Connection Lifecycle

```
Client                              Server
  │                                     │
  │──── Hello ───────────────────────>│  capabilities + auth
  │<─── Welcome ──────────────────────│  session_id + codec
  │   [optional Resume exchange]        │
  │<─── Ready ───────────────────────│  limits + heartbeat
  │                                     │
  │ ═══ bidirectional data frames ════  │
  │                                     │
  │──── Close ───────────────────────>│  (either side)
```

## Repositories

| Repository | Description |
|-----------|-------------|
| [rifts](https://github.com/rift-proto/rifts) | Rust server implementation. Standalone WebSocket + framework adapters (Axum, Actix Web, Warp, Ntex). |
| [rift-ts](https://github.com/rift-proto/rift-ts) | TypeScript client SDK for browser and Node.js. |

## Protocol Highlights

- **Typed frame envelope** — single `Frame` structure for all wire traffic, with priority (6 levels), 10 flag bits, and routing/correlation metadata
- **CBOR + JSON codec negotiation** — binary encoding for production, JSON for debugging, negotiated once at handshake
- **8 message classes** — Event, Command (RPC), Reply, State (latest-only), Datagram (best-effort), Stream (ordered segments), Snapshot, System
- **Topic profiles** — per-topic retention policy (None / TTL / Count / Size / Durable / Latest), ordering policy (None / Connection / Publisher / Topic / Key / Causal), and rate limits
- **Session resume** — transport-agnostic logical sessions with epoch validation and per-topic offset replay
- **Acknowledgement system** — 9 status levels (Received → Processed), 6 policies (None → Application)
- **Flow control** — backpressure with 6 strategies (Pause / DropVolatile / CoalesceState / Downgrade / Disconnect / SnapshotLater) + token-bucket rate limiter
- **Heartbeat** — ping/pong with jitter to prevent thundering-herd, configurable missed-pong threshold
- **30 error codes** and **16 close codes** — structured, machine-readable, with retryability semantics

## TPL Specifications

The complete Rift/1 wire protocol is specified as **TPLs** (Technical Protocol Listings) in the [`tpl/`](tpl/) directory. See [`tpl/README.adoc`](tpl/README.adoc) for the reading guide.

| TPL | Title | Status |
|-----|-------|--------|
| [TPL-FRAME-1](tpl/TPL-FRAME-1.adoc) | Frame Envelope | `mandatory` |
| [TPL-CODEC-1](tpl/TPL-CODEC-1.adoc) | Codec Negotiation | `mandatory` |
| [TPL-MSG-1](tpl/TPL-MSG-1.adoc) | Message Types | `mandatory` |
| [TPL-TOPIC-1](tpl/TPL-TOPIC-1.adoc) | Topic Profiles | `mandatory` |
| [TPL-SUB-1](tpl/TPL-SUB-1.adoc) | Subscribe Modes | `mandatory` |
| [TPL-ACK-1](tpl/TPL-ACK-1.adoc) | Acknowledgement System | `mandatory` |
| [TPL-SESSION-1](tpl/TPL-SESSION-1.adoc) | Session & Resume | `mandatory` |
| [TPL-HANDSHAKE-1](tpl/TPL-HANDSHAKE-1.adoc) | Hello / Welcome / Ready Handshake | `mandatory` |
| [TPL-AUTH-1](tpl/TPL-AUTH-1.adoc) | Authentication | `mandatory` |
| [TPL-FLOW-1](tpl/TPL-FLOW-1.adoc) | Flow Control | `mandatory` |
| [TPL-HEARTBEAT-1](tpl/TPL-HEARTBEAT-1.adoc) | Heartbeat | `mandatory` |
| [TPL-ERROR-1](tpl/TPL-ERROR-1.adoc) | Error Codes | `mandatory` |
| [TPL-CLOSE-1](tpl/TPL-CLOSE-1.adoc) | Close Codes | `mandatory` |
| [TPL-VERSION-1](tpl/TPL-VERSION-1.adoc) | Protocol Versioning | `mandatory` |
| [TPL-CONFIG-1](tpl/TPL-CONFIG-1.adoc) | Server Configuration | `optional` |

## License

Dual-licensed under MIT or Apache-2.0.
