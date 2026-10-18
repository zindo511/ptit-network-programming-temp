# ExamLink — Project Proposal (M1)

> Draft for team review, 2026-10-19. Intended behavior; no implementation or test results are claimed.

## 1. Team & Topic

**Members:** Trịnh Quang Huy, Nguyễn Thanh Bằng, Trịnh Minh Đức (see [README](../README.md)).
**Topic:** Online multiple-choice exams with JavaFX clients, a Java TCP socket server and session recovery after connection loss.

## 2. Problem

Connection loss leaves candidates unsure whether their latest answer or submission was saved. Proctors need to distinguish lost contact from exam completion. Reconnecting alone cannot resolve this: the application must retain exam state and confirm what it accepted. ExamLink studies this recovery mechanism, without claiming that existing exam platforms lack it.

## 3. Idea

The server maintains each candidate's exam session independently of the socket. A JavaFX client retains a token in RAM and uses it to resume, synchronizing saved answers, versions, exam status and the unchanged deadline. Save confirmation means acceptance into server RAM, not durable storage. Proctors use a client with server-checked `PROCTOR` permissions to monitor their room; the server needs no GUI.

## 4. Scope

| Required MVP | Optional extensions | Outside MVP |
|---|---|---|
| Authentication, room entry, receiving questions | Richer incident history and filters | Database; recovery after server restart |
| Answer saving, confirmation and token-based resume | Practice mode with individual start times | Durable client storage; app-restart recovery |
| Manual/timeout submission; proctor status; heartbeat | Additional exam-authoring tools | Web UI; webcam; essays; distributed servers |

Recovery assumes the server remains running and the client retains its token. Network loss does not pause the exam; timeout submission uses server-accepted answers. Initial exam provisioning and score visibility need team agreement.

## 5. Key Networking Challenge

- **Saving and retries:** TCP ACK is not a save confirmation. Correlate application responses; use answer versions to prevent stale overwrites and reject conflicting reuse. Repeated submission returns the recorded outcome.
- **Reconnect races:** Replace the active connection identity under the session lock, then close the old socket. Old handlers must not change state or mark the new connection offline.
- **Last-second races:** Saving, submission and expiry share a per-session lock. Check deadlines during request handling as well as scheduled checks. Keep network I/O outside the lock.

## 6. Initial Design Sketch

- **Stack:** Java 21, JavaFX, Maven and Java socket APIs without Spring. Client and server communicate only through the documented protocol.
- **Protocol:** TCP with length-prefix framing. Handle partial reads and multiple frames per read. Prefix width, payload format/encoding, size limits and exact wire schemas remain M2 decisions.
- **Server:** One acceptor; one virtual-thread handler task per connection; one shared scheduler for deadlines and heartbeat timeouts. A concurrent registry locates sessions; individual session locks protect compound updates. Virtual threads do not remove resource limits.
- **Client:** Network I/O runs off the JavaFX UI thread, with one reader and coordinated ordered writes per connection. Match responses to the current answer version before showing “Saved”.
- **Time/status:** Store a deadline per session; client countdown is display only. Separate `ONLINE/OFFLINE` from exam status and submission reason. `PING/PONG` heartbeat detects lost contact, not its cause; timeout values are not yet fixed.
- **Deployment:** Target `docker compose up --build` for the server and native JavaFX clients. Addresses/ports come from environment configuration. Bind to `0.0.0.0` in Docker; container clients use `server`, native clients use the reachable host address.

Initial operations below are responsibilities, **not final wire command names**:

| Operation | Direction | Purpose |
|---|---|---|
| Authenticate / join | Client → server | Validate identity/role and obtain a permitted exam session |
| Save answer | Client → server | Validate and acknowledge an answer version |
| Resume | Client → server | Validate token, replace connection and return saved state |
| Submit | Client → server | Finalize once and return the outcome |
| Monitor room | Proctor ↔ server | Obtain a snapshot and authorized status updates |

Specify commands, heartbeat, errors and sequences in [protocol-design.md](protocol-design.md), and locking/lifecycle in [architecture.md](architecture.md), before implementation. Both documents currently remain starter templates.

## 7. Alternatives Considered

| Choice | Alternative | Reason / trade-off |
|---|---|---|
| TCP | UDP | Built-in ordered, reliable transport; framing and application confirmations still needed |
| Length-prefix framing | Newline-delimited (`\n`) framing | No delimiter-specific escaping for multiline content; payload length is known before buffer allocation. Validate the length against a maximum before allocating; payload-format escaping (e.g. JSON) still applies. Less readable in raw text tools than newline-delimited messages. |
| JavaFX | Browser / Electron | Direct Java socket access and one language; requires desktop distribution |
| Java without Spring | Spring Boot / Integration | Explicit networking for learning; manual component/lifecycle management |
| Virtual threads | Fixed blocking pool / NIO | Straightforward blocking handlers; capacity still requires testing |
| RAM sessions | Database | Smaller MVP; server restart loses state |

## 8. Ownership Plan

This is the work allocation set by the team leader for M1, not a record of completed contributions. Each owner implements, tests and documents their component. All members should review the scope and coordinate interfaces before implementation; individual acceptance and completed work are not claimed here.

| Member | Planned responsibility |
|---|---|
| Trịnh Quang Huy — team leader | Server exam-session and business logic: token validation, answer versions/retries, deadlines, submission, per-session locking and active-connection replacement on resume. Own the corresponding race-condition tests and state/lifecycle documentation; coordinate interface agreement, integration and reviews with the team. |
| Nguyễn Thanh Bằng | Protocol and server networking: framing/codec, acceptor, virtual-thread handlers, coordinated writes, heartbeat detection, connection cleanup and Docker. Test partial/coalesced/malformed frames, timeouts and concurrent connections; maintain protocol and deployment documentation. |
| Trịnh Minh Đức | JavaFX client: candidate/proctor screens, client TCP transport, response correlation, reconnect, countdown display and save status. Test UI responsiveness, stale responses and recovery flows; document client setup and usage. |

Networking handlers decode and dispatch requests to session/business logic, then send responses outside session locks. On reconnect, business logic authorizes the replacement and networking closes the old socket; stale-handler cleanup must respect the active connection identity. All three members agree the protocol and review end-to-end behavior together.

## 9. Risks & Validation Plan

| Risk | Planned check / mitigation |
|---|---|
| Save reply lost; stale retry | Reconnect and retry; verify accepted versions and answer consistency |
| Reconnect or submission/expiry races | Exercise competing handlers; verify one active connection and one final outcome |
| Malformed frames, silence, slow clients | Bound frames/connections/queues, apply timeouts, isolate failures |
| Unproven capacity | Run concurrent-client tests and record actual resource use and results |

These are planned checks, not evidence. Record results in [testing-guide.md](testing-guide.md). Settle deadline-boundary policy, token lifecycle, version scope and heartbeat thresholds during M2.

## 10. Instructor Feedback

Pending M1 review; record feedback when received.
