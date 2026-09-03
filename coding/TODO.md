# Future Coding TODO — not yet authorized

After the Design Gate, likely implementation work includes:

- [ ] Create normative WorkflowSpec / TaskSpec schemas.
- [ ] Create WorkflowRun / TaskRun / TaskAttempt domain types.
- [ ] Implement validation and cycle detection.
- [ ] Implement execution snapshot resolution/digesting.
- [ ] Implement durable state storage.
- [ ] Implement event journal/outbox.
- [ ] Implement dependency evaluator.
- [ ] Implement reconcile workers.
- [ ] Implement retry/timer service.
- [ ] Implement finalizer scheduling.
- [ ] Implement redrive/restart service.
- [ ] Define execution adapter SDK/protocol.
- [ ] Define persistent-service execution adapter contract.
- [ ] Define asynchronous submit/observe/callback protocol.
- [ ] Define Capability-to-ExecutionBinding resolver port.
- [ ] Implement logical dispatch admission/backpressure after semantics are approved.
- [ ] Implement one reference adapter only after boundary tests exist.
- [ ] Implement API/CLI.
- [ ] Implement Run Graph/Timeline UI.
- [ ] Add conformance tests for state/dependency/retry/cancel semantics.
- [ ] Add fault-injection tests for controller crash and duplicate dispatch.

This list is intentionally implementation-agnostic; language/database/runtime choices are not Phase-0 decisions unless required by semantics.
