# 8. Realtime events and cancellation

The realtime design is an external contract. It defines observably what a client
sees; it does not prescribe how the daemon stores or schedules events.

## 8.1 Event stream operation

- RTS-001: **The daemon MUST expose the event stream as one long-lived operation
  that the client can open over any transport.** `[current]`
- RTS-002: **The stream request MUST select one or more channels.** `[current]`
- RTS-003: **The stream request MUST carry a cursor; zero MUST mean "no
  cursor".** `[current]`
- RTS-004: **A stream that selects no channel MUST deliver no channel's
  events.** `[current]`
- RTS-005: **The documented channels MUST cover daemon-wide events, events for
  one session, events for one task, and events for one workflow run.** The
  channel naming scheme is open (`OQ-119`). `[current]`
- RTS-006: **The daemon MUST treat a channel named twice in one selection as one
  channel.** `[current]`
- RTS-007: **The daemon MUST refuse a request that presents a cursor the event
  window can no longer serve, before any event is delivered.** `[current]`

## 8.2 Events and identifiers

- RTS-008: **Every event MUST carry a daemon-wide monotonic identifier, a kind, a
  channel, and a payload.** `[current]`
- RTS-009: **The identifier space MUST be one counter across all channels,
  starting at 1, and MUST NOT decrease within a daemon lifetime.** `[current]`
- RTS-010: **The event kinds MUST be unspecified, snapshot, heartbeat, and
  ordinary event.** Unspecified is the contract's "cannot act on this" value.
  `[current]`
- RTS-043: **A receiver MUST treat an event kind it does not know as unspecified
  and MUST NOT act on it.** `[current]`
- RTS-011: **The daemon MUST bound the set of events from which a cursor can
  resume, and MUST discard the oldest events when the bound is reached.**
  `[current]`
- RTS-012: **The event window MUST NOT be durable. A daemon restart MUST begin
  the identifier space again and MUST expire every cursor.** `[current]`
- RTS-013: **The event stream MUST NOT be treated as the source of truth for
  application state.** `[current]`
- RTS-014: **The event payload MUST remain opaque to the stream machinery, so a
  change to its encoding is a breaking contract change governed by `CMP-004`
  rather than a change to the stream.** The payload encoding is open (`OQ-119`).
  `[current]`

## 8.3 Stream opening: snapshot and resume

- RTS-015: **A stream opened with no cursor MUST begin with a snapshot message.**
  `[current]`
- RTS-016: **The snapshot MUST consist of one message for each selected channel
  that holds an event, in identifier order, and each message MUST carry that
  channel's head event.** A selected channel with nothing in the window
  contributes no snapshot message. `[current]`
- RTS-017: **When no selected channel holds an event, the stream MUST open with
  one empty snapshot message that names no channel and carries the daemon's
  latest identifier.** `[current]`
- RTS-018: **A stream opened with a servable non-zero cursor MUST begin with the
  events that follow that cursor, in identifier order, typed as ordinary events,
  and MUST NOT carry a snapshot.** `[current]`
- RTS-019: **The daemon MUST open the live subscription before it reads the
  window, so an event published between the two is not lost.** `[current]`
- RTS-020: **The daemon MUST suppress live events whose identifiers were already
  delivered by the opening messages.** `[current]`
- RTS-021: **A heartbeat MUST travel as an ordinary stream message and MUST carry
  the daemon's latest identifier.** `[current]`
- RTS-022: **The daemon MUST send heartbeat messages on an open stream.** The
  current interval is 15 seconds; the required interval and whether it is
  configurable are open (`OQ-066`). `[current]`

## 8.4 Cursor semantics

- RTS-023: **A cursor of zero MUST never be treated as expired.** `[current]`
- RTS-024: **The daemon MUST treat a non-zero cursor as servable exactly when the
  cursor is not above the latest identifier and is not more than one below the
  oldest resumable position.** `[current]`
- RTS-025: **A cursor below the window MUST be refused as expired.** `[current]`
- RTS-026: **A cursor above the latest identifier MUST be refused as expired,
  which is how a daemon restart expires every cursor.** `[current]`
- RTS-027: **An expired-cursor refusal MUST be distinguishable by the client from
  every other stream failure, without parsing message text.** `[current]`
- RTS-028: **The client MUST answer an expired cursor with a resync only when its
  caller chooses to; the client MUST NOT retry or resync automatically.**
  `[current]`
- RTS-029: **A caller's cursor MUST be the highest ordinary event identifier it
  has processed.** `[current]`
- RTS-041: **A snapshot MUST advance the caller's cursor only for the channels
  that the snapshot message names.** `[current]`
- RTS-042: **A heartbeat MUST NOT advance the caller's cursor, even though it
  carries the daemon's latest identifier.** `[current]`
- RTS-030: **A caller that changes its channel selection while resuming MUST
  resync rather than resume, because its held state is for other channels.**
  `[current]`

## 8.5 Ordering and delivery

- RTS-031: **The daemon MUST ensure that a stream's observed order equals
  event-identifier order across all its selected channels.** `[current]`
- RTS-032: **The client MUST receive events in increasing identifier order and
  MUST NOT need to sort or discard events to repair ordering.** `[current]`
- RTS-033: **The daemon MUST deliver every event it accepts for a stream at most
  once on that stream.** `[current]`

## 8.6 Slow consumers

- RTS-034: **The daemon MUST apply bounded backpressure to a stream consumer that
  stops reading, and MUST NOT silently skip events.** `[current]`
- RTS-035: **When a consumer falls behind the daemon's delivery window, the daemon
  MUST end that stream with a terminal slow-consumer refusal that names the loss
  and carries no further events.** `[current]`
- RTS-036: **The client MUST report a slow-consumer drop as a distinct typed
  error, and the documented answer MUST be a cursorless resync.** `[current]`
- RTS-037: **A dropped slow consumer MUST NOT be mistaken for a transport
  failure.** `[current]`

## 8.7 Request identifiers, correlation, and cancellation naming

- CAN-001: **A client call MAY declare an opaque request identifier; a call that
  declares none MUST be served as before, and it is not cancellable.**
  `[current]`
- CAN-002: **A request identifier MUST be at most 64 printable characters,
  with no space, so that it is safe to carry with a call.** Its form is open
  (`OQ-123`). `[current]`
- CAN-003: **The client MUST generate a fresh identifier for each stream it opens,
  so that two streams from one client never share an identifier.** `[current]`
- CAN-004: **The daemon MUST echo a declared request identifier verbatim in the
  call's answer.** `[current]`
- CAN-005: **The daemon MUST echo a declared request identifier verbatim,
  including a malformed one, and MUST NOT refuse a call only because the
  identifier is malformed.** `[current]`
- CAN-006: **The daemon MUST validate a request identifier on the cancellation
  call and on a stream request that registers in-flight work; a malformed
  identifier on either MUST be a typed invalid-argument refusal.** Every other
  call is served and the identifier is echoed verbatim. `[current]`
- CAN-007: **A request identifier MUST NOT be reused as the plugin request
  identifier; the two identifier spaces are distinct.** `[current]`

## 8.8 Cancellation

- CAN-008: **The daemon MUST expose an explicit cancellation operation that names
  the target request identifier.** `[current]`
- CAN-009: **The cancellation operation MUST be idempotent.** `[current]`
- CAN-010: **A cancellation MUST report one of two outcomes: cancelled, meaning an
  in-flight request under that identifier was found and is being stopped, or
  already finished, meaning none was in flight.** `[current]`
- CAN-011: **Already finished MUST cover work that finished, work already
  cancelled, and work never seen, which is what makes a second call
  idempotent.** `[current]`
- CAN-012: **A cancellation MUST stop only work the calling surface started; a
  cancellation naming another surface's request MUST answer already finished and
  MUST NOT touch that work.** `[current]`
- CAN-013: **A cancellation for an identifier that is not in flight MUST answer
  already finished without entering plugin code.** `[current]`
- CAN-014: **Dropping or disconnecting a stream MUST NOT cancel any work.**
  `[current]`
- CAN-015: **Closing a user interface MUST NOT cancel work (see `PRD-C1`).**
  `[current]`
- CAN-016: **Cancelling a stream's request MUST end that stream with a distinct
  cancelled refusal after which no further events are delivered.** `[current]`
- CAN-017: **The client MUST report a cancelled stream as a distinct typed
  error.** `[current]`
- CAN-018: **Cancellation MUST stop future work and MUST NOT undo external effects
  the work already performed or events it already published.** `[current]`
- CAN-019: **The daemon's in-flight registry MUST be in memory; a restart
  forgets in-flight work, and a cancellation for a pre-restart identifier MUST
  answer already finished.** `[current]`
- CAN-020: **A second registration under the same surface and identifier MUST
  replace the earlier entry, and a completed or dropped registration MUST remove
  only its own entry.** `[current]`
- CAN-021: **A stream registration that is refused for an expired cursor MUST
  leave nothing in flight.** `[current]`
- CAN-022: **A cancelled stream's end signal MUST say that the stream stopped; it
  MUST NOT claim that the work was undone.** `[current]`

## 8.9 Stream behaviour under failure

- RTS-038: **A transport failure on a stream MUST remain a transport error and
  MUST NOT be reported as a deliberate drop or a cancellation.** `[current]`
- RTS-039: **No event MUST be delivered after a terminal stream refusal.**
  `[current]`
- RTS-040: **A caller that cannot keep up MUST be able to tell that its view is
  stale, because the daemon ends the stream instead of continuing with a gap.**
  `[current]`
