# Mobile Money SIM Swap Protection

How a **mobile money platform** can stop fraud that starts with a SIM swap: the attacker gets a replacement SIM for the victim's number, receives the OTPs, resets the PIN, and drains the wallet. This repo covers the signals, the rules, and the customer-recovery path. It is part of a set of educational notes from [MoboGage](https://evdsystem.com/about-mobogage/), the team behind [EVD System](https://evdsystem.com/).

---

## The attack in plain terms

Mobile money identity is tied to the MSISDN. If a fraudster convinces a retail shop or a call centre to issue a new SIM for that number, the old SIM goes dead and every SMS, USSD session and OTP now reaches the attacker. On USSD-first markets this is often enough to reset a PIN with a few personal details.

The victim usually notices because their phone shows "No service". By then the first transfers may already be out.

## Signals worth wiring in

**SIM change timestamp.** Most operators can expose the last SIM change date for an MSISDN through an internal API or a GSMA-style CAMARA SIM Swap API. This is the single most useful signal.

**IMSI change.** The subscriber identity on the network changes when the SIM changes. If the wallet stores the IMSI it saw at enrolment, a mismatch is a strong flag.

**Device change.** For app users, a new device binding at the same time as a PIN reset is a classic pattern.

**Behaviour after reset.** A PIN reset followed within minutes by a max-value transfer to a never-seen recipient, or a cash-out at an agent far from the usual area.

## Rules that hold up in production

| Situation | Suggested response |
|-----------|-------------------|
| PIN reset requested, SIM changed recently | Refuse self-service reset, route to agent or branch with ID |
| Login from new device, SIM changed recently | Allow balance view only, block outbound value for a cooling period |
| High-value transfer after recent SIM change | Hold and call back on a secondary contact, or require agent KYC |
| SIM changed long ago, normal behaviour | No extra friction |

The cooling period and value thresholds are business choices. Pick them with the fraud team and revisit them once you have a few months of data.

## Worked example (figures are illustrative)

At 10:02 a customer's SIM is replaced at a retail shop. At 10:09 a USSD PIN reset is attempted. The platform queries SIM change time, sees it was seven minutes ago, and declines the self-service reset with a message to visit an agent with ID. At 10:20 the real customer calls the care line from a relative's phone because their own phone lost service. Care flags the MSISDN, the wallet is put on `FRAUD_HOLD`, and the operator reverses the SIM change after identity checks. No value left the wallet.

Without the SIM change check, the same timeline ends with a reset at 10:09 and a transfer at 10:11.

## What to log

`msisdn`, `sim_change_at` as returned by the operator, `imsi_seen`, `device_id`, `pin_reset_requested_at`, `decision` (ALLOW, STEP_UP, DENY), `decision_reason`, and the case id if care got involved. Keep the operator API response itself, because disputes later turn on what the platform knew at that moment.

## Customer recovery

When a swap fraud does get through, the customer needs one owner for the case. The usual sequence is freeze the wallet, freeze recipient wallets if they are on-net, trace cash-outs to agents, and open a reversal request for anything still in the system. The [mobile money dispute resolution](https://github.com/iwtglobal/mobile-money-dispute-resolution) repo covers the case workflow.

A short control checklist is in [docs/sim-swap-controls.md](./docs/sim-swap-controls.md).

---

## FAQ

**What is SIM swap fraud in mobile money?**
It is when an attacker gets a new SIM issued for the victim's number, receives their OTPs and USSD sessions, resets the wallet PIN and moves the money out.

**How can a mobile money platform detect a SIM swap?**
By checking the last SIM change time from the mobile operator before PIN resets and high-value transactions, and by comparing the IMSI and device seen at enrolment.

**Should every SIM change block the wallet?**
No. Legitimate SIM replacements are common. Add friction only for a period after the change and only for risky actions such as PIN reset or large transfers.

**Does this apply to agent banking too?**
Yes. Agent cash-out right after a SIM change is a frequent pattern, so the agent app should show the hold reason instead of a generic error.

---

## Further reading

- [Mobile money platform with merchant payment](https://evdsystem.com/mobile-money-platform-with-merchant-payment/) on evdsystem.com
- [Wallet cash-in, cash-out, transfers and reversals](https://evdsystem.com/evd-software/wallet-cash-in-out-transfers-and-reversals/) on evdsystem.com
- Related repo: [mobile-money-fraud-monitoring](https://github.com/iwtglobal/mobile-money-fraud-monitoring)

## License

MIT. See [LICENSE](./LICENSE). Educational material only.
