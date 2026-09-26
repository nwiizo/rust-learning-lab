# Reading Rust across system boundaries

English | [日本語](systems-thinking_ja.md)

Use for Rust services, persistence, concurrent state, retries, asynchronous work, or questions about what
types cannot establish. Select the relevant boundary; do not turn a syntax answer into a distributed-systems course.
These are engineering perspectives applied to Rust, not additional language guarantees.

## Locate the boundary of each guarantee

Ask which value, process, storage operation, and time interval a statement covers.

| Rust evidence | What it supports | What still needs evidence |
|---|---|---|
| `&mut Client` | Exclusive access through that borrow, subject to Rust's aliasing rules | Exclusive access to a database row through other clients, processes, or replicas |
| `Arc<Mutex<State>>` | Shared ownership and mutually exclusive access by users of this mutex | Atomic multi-step workflows, deadlock freedom, bounded waiting, or coordination across machines |
| `Send` / `Sync` | Safe transfer across threads / safe sharing of references across threads, assuming sound implementations | Business invariants, transaction isolation, ordering of external effects, or eventual completion |
| `Result<Receipt, Error>` and `?` | A represented success/error path and propagation to the enclosing return boundary | Whether a remote operation committed before the error, or whether earlier effects were undone |
| A guard and `Drop` | Cleanup specified by the type when destruction runs | Successful remote rollback, durable commit, cleanup after process termination, or an observable cleanup error |

For the exact thread properties, use the standard [Send](https://doc.rust-lang.org/std/marker/trait.Send.html)
and [Sync](https://doc.rust-lang.org/std/marker/trait.Sync.html) definitions. An `Arc` does not make arbitrary
contents thread-safe. [Drop](https://doc.rust-lang.org/std/ops/trait.Drop.html) describes destruction, not a
general transaction mechanism.

An invariant is a condition that must continue to hold, such as “at most one active reservation per slot.”
First state that condition; then identify the mechanism covering every writer. Two safe Rust processes
can both read “free” and both attempt to reserve it. A mutex inside each process does not coordinate them.
A database uniqueness constraint, atomic conditional update, or suitable transaction can provide the needed
boundary. Select one from the actual invariant and database behavior, not the Rust keyword used by the client.

Do not infer an isolation level from `begin()` or a transaction type. For example, PostgreSQL documents
different anomalies at different [isolation levels](https://www.postgresql.org/docs/current/transaction-iso.html).
When serialization conflicts require a retry, retry the complete transaction's decision, not just its last
write using values read earlier. A committed database transaction does not automatically include an email
or another service's operation.

## Treat uncertainty as information, not a generic failure

Imagine submitting a job to an external service. The caller's observation and the remote state can differ:

| Observation | Possible remote state | Next decision |
|---|---|---|
| Local validation rejects the request before sending | No submission from this attempt | Correct the input |
| Service explicitly rejects the request under its protocol | Rejected as specified by that response | Explain rejection; check whether retrying can change it |
| A documented completion receipt arrives | Completed to the extent promised by that receipt | Preserve the receipt and the promise's scope |
| Timeout or connection loss after sending | Not received, still running, or already completed | Reconcile by request identity or use a documented safe retry mechanism |

An error model may distinguish `Rejected` from `OutcomeUnknown`. This is an API design choice, not a rule
that every Rust error needs those variants. Explain why the caller needs the distinction before adding types.
Do not use `?` to erase information the caller needs for recovery. The
[side-effect example](worked-examples.md#an-error-return-does-not-undo-a-side-effect) shows the smaller local
fact: returning `Err` does not reverse a mutation already performed.

Idempotence means repeating an operation has the same intended effect as applying it once. Ownership of
one request value or a `FnOnce` bound does not establish this across retries, deserialization, or restarts.
An idempotency key is useful only with a receiving protocol that enforces it. Keep the same key for the
same logical operation, check how a reused key with different contents is rejected, and specify retention.
If the deduplication record and state change can be committed independently, inspect the crash gap between
them. An in-memory `HashSet` demonstration does not establish durable deduplication. See the engineering
account of [idempotent retries](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/).

If storing an intended event with a database change, a
[transactional outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)
can close that local write gap.
It still needs a delivery worker and consumer-side handling of duplicate delivery. This is a conditional
design option, not a requirement for every application or a claim of automatic exactly-once effects.
Ask “once at which boundary, and under which failure assumptions?” before using that phrase.

## Trace suspension, ordering, and load

### Cancellation and unfinished work

At an `.await` that can suspend, identify what has already changed, what is retained by the future,
and what remains to finish. A cancelled future can release local guards while leaving an external effect
in place. Review the operation's cancellation documentation and recovery path; memory safety alone is
not evidence that the application can safely discard progress.

The future being awaited may be a handle to other work. Tokio's
[JoinHandle](https://docs.rs/tokio/latest/tokio/task/struct.JoinHandle.html) detaches its task when dropped;
dropping the handle does not cancel that task. Therefore a timeout around an owned handle and a timeout
around the actual operation are not equivalent. Aborting local work also does not roll back a request
already accepted elsewhere. Do not introduce a runtime dependency solely to explain this distinction.

For shutdown, distinguish stopping new work, finishing or durably recording accepted work, and reporting
completion. `Drop` alone is not evidence that all asynchronous work finished. Returning early with `?`
or cancelling at a suspension point deserves the same question: which state remains observable?

### Time and visibility

Use [Instant](https://doc.rust-lang.org/std/time/struct.Instant.html) for local elapsed time and deadlines;
it is not a shared timestamp between machines or a persisted ordering scheme. A wall-clock timestamp is
not proof of causal order. Distinguish event time, receipt time, and processing time when they matter.

A successful write followed by a read from a lagging replica or a cache can return older data. An `&T`
keeps access to a valid value; it does not make that value current. If an API exposes a version, explain
how conditional writes or a documented read policy use it. Merely wrapping it in `Version(u64)` supplies
no consistency protocol. Likewise, a sequence is ordered only within its documented scope, such as one
partition; multiple partitions and parallel consumers do not imply one global execution order.

Distinguish serializability (transactions behave like some serial ordering) from linearizability (operations
respect real-time precedence as well). A local lock, logical timestamp, or generated ID is not, by itself,
evidence that a distributed service provides either property. For a concrete API's scope and exceptions,
inspect its own guarantees, such as [etcd's operation and watch distinctions](https://etcd.io/docs/v3.6/learning/api_guarantees/).

### Backpressure and cost

A bounded queue controls how much it buffers; the full-queue policy still matters. Tokio's
[channel guide](https://tokio.rs/tokio/tutorial/channels) explains awaiting capacity. Rejecting, waiting,
or discarding work have different user-visible effects. Capacity limits messages, not necessarily bytes.
Unbounded numbers of tasks waiting to send can still retain large request bodies outside that queue.

Measure the actual workload: data sizes, concurrent requests, hot keys, and slow dependencies. Separate
service time from time spent waiting in a queue. Report a distribution such as p50/p95/p99 together with
load and error rate; a fast average or a benchmark of only successful requests can hide overload.
Retries can add load exactly when capacity is scarce, so relate retry budgets and deadlines to admission
control instead of treating a retry loop as a universal reliability improvement.

## Follow data through representation and time

### Choose a representation from the operations it must support

An owned tree, a graph addressed by IDs, a table, and an event sequence expose different operations.
Do not force every relationship into nested ownership or fix every cycle with `Rc<RefCell<_>>`.
IDs can simplify graph updates, but looking up a removed ID must have defined behavior. An `enum` can
represent alternatives explicitly, while a struct groups values that coexist. Neither choice alone
defines database constraints or the wire format.

For in-memory collections, start with required lookup, iteration, ordering, and mutation behavior.
A `HashMap` does not promise a stable iteration order; a `BTreeMap` supports key ordering. Neither choice
reproduces the durability or replication properties of a database index. Likewise, a borrowed slice may
avoid a copy within one process, but crossing a network still needs an encoding and an ownership decision
for buffers. Investigate the dominant cost before converting a whole API to lifetimes or shared ownership.

### Parse, validate, and evolve separately

Reading bytes into a Rust type establishes only the conditions enforced by that reader. A deserialized
`String` is not automatically a valid account name; a `u64` is not automatically the intended unit.
Use a validated constructor or `TryFrom` when invalid values would otherwise spread. Preserve that check
on all construction paths, including deserialization; private fields alone do not make a derived decoder
call a validating constructor. Serde supports fallible conversion through
[`try_from`](https://serde.rs/container-attrs.html).

Data can outlive the executable that wrote it. Test old bytes with the new reader and, when rolling
deployment or rollback requires it, new bytes with the old reader. A Rust API change that compiles for
every caller in this workspace can still break a stored record or an independently deployed client.
[`serde(default)`](https://serde.rs/field-attrs.html) supplies missing values; decide whether the default
has the intended meaning. [`deny_unknown_fields`](https://serde.rs/container-attrs.html) rejects unknown
fields and may conflict with accepting future additions. Missing, null, and an explicit default are not
automatically interchangeable. Match these choices to the actual format and versioned examples.

### Rebuild derived state without repeating external actions

Separating a transformation from I/O can make it possible to replay fixed input and compare outputs.
An iterator pipeline is not necessarily pure: closures can read clocks, use randomness, or perform I/O.
Identify those dependencies, processing order, and the code/schema version needed to reproduce a result.
Do not silently resend notifications when rebuilding a cache or replaying stored events.

For a consumer, record how output and progress relate. Saving progress first can skip unfinished output
after a crash; saving output first can repeat it after a restart. A transaction spanning both, or a
documented idempotent output protocol, may address that gap. The correct mechanism depends on the stores
involved; a local `fold` does not imply a distributed recovery protocol.

## Verify the claim at the layer where it can fail

| Claim | Focused evidence | What the evidence does not establish |
|---|---|---|
| A domain type rejects an invalid value | Constructor tests and inspection of every public construction path | Validity of every record already persisted |
| Concurrent reservations preserve uniqueness | Competing requests against the real database constraints and isolation level | Every failure/recovery scenario |
| Retry preserves one logical effect | Lost-response, duplicate, and restart cases; inspect stored effects | A guarantee from the retry count alone |
| A format change is compatible | Saved old/new payloads tested against the required reader versions | Compatibility from a same-version round trip alone |
| A service stays usable under load | Queue/task/memory measurements, tail latency, rejection rate, and recovery after overload | Performance from compilation, async syntax, or one microbenchmark |

Model tests and controlled fakes help make a failure repeatable; they are not evidence of a real store's
isolation or durability. Test the boundary whose behavior the claim depends on. A status message should
distinguish “accepted” from “completed,” and “failed” from “outcome not yet known” when those distinctions
are real. Expose stable recovery information while avoiding sensitive payloads in `Debug` or error logs.

Data retention, deletion of derived copies, authorization, and explaining an automated decision are
product responsibilities. Ownership determines who manages a Rust value's resources; it does not
determine who is entitled to use a person's data. Bring in these concerns when the actual data flow
requires them, without turning every small Rust example into a production-system checklist.
