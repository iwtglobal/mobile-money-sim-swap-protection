# SIM Swap Controls Checklist

## Integration

- [ ] Operator SIM change API connected (internal or CAMARA SIM Swap)
- [ ] Timeout behaviour defined: if the API does not answer, treat risky actions as STEP_UP, not ALLOW
- [ ] IMSI captured at enrolment where the network allows it
- [ ] Device binding for app channel

## Decision points

- [ ] PIN reset (USSD, app, care)
- [ ] New device login
- [ ] Transfers above threshold
- [ ] Agent cash-out above threshold
- [ ] Change of linked bank account or beneficiary

## Operations

- [ ] Care line can place `FRAUD_HOLD` in one step
- [ ] Agent app shows a clear hold reason
- [ ] Every decision logged with the raw operator response
- [ ] Monthly review of false positives with the fraud team
