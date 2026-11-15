# Keep authorized work moving across usage resets

Provenance: explicit owner preference, recorded 2026-11-15 after an unattended build
stopped at phase boundaries despite available allowance. This is a working preference,
not a claim that every agent product supports usage inspection or scheduled resumption.

## A completed phase is a checkpoint, not a reason to become idle

**Symptom.** A build has permission, credentials, a clear next step and remaining usage,
but stops after one phase and waits for the next scheduled wake or another human prompt.

**Cause.** The agent treats "work in phases" as "end the turn after each phase". An hourly
follow-up disguises unnecessary idle time as automatic continuation.

**Fix.** Continue through successive safe, approved phases in the active turn. Verify,
commit and checkpoint between phases; then begin the next one without asking again.
Periodically check actual account usage, especially before substantial work and after
expensive phases. Do not spend the user's quota merely to exhaust it; do useful work.

Stop only at actual completion, an explicit human stop, insufficient usage reserve, or
a concrete blocker requiring new authority, missing information or an external change.
Persisting toward completion never expands authorization for credentials, spending,
public exposure, destructive changes or production-impacting actions.

**Why it was hard to find.** Each phase genuinely passed its tests, and a continuation
automation existed. Neither fact proved that the agent kept working while it could.

## Low allowance: checkpoint and schedule the next eligible resume

1. Use the product's account-usage tool when available. Report remaining allowance,
   window and reset timestamp; percentages are not an exact token count. Missing data
   means unknown, not zero or unlimited.
2. Keep enough reserve to finish the current safe unit, save evidence and schedule
   resumption. Default planning guardrails: checkpoint near 5% short-window remaining;
   resume with at least 15% short-window and 10% weekly remaining, unless the owner sets
   different thresholds. Recheck every limiting window; one reset may not unblock work.
3. Save branch, commits, dirty-file ownership, tests, exact next steps, pending jobs,
   approval boundaries and blockers in the project. Never save plaintext secrets.
4. Update an existing continuation automation rather than creating a duplicate. Set
   the next eligible wake just after the relevant reset, using the owner's timezone.
   Check live allowance again on wake; do not assume a reset restored every limit.
5. Resume directly from the checkpoint. Do not restart finished work, duplicate uploads,
   re-run expensive tests without cause, or ask again for already recorded approvals.
6. Once allowance is adequate, continue actively across phases; a fallback heartbeat
   is not a reason to wait between them. Notify only meaningful progress, failures or
   needed user action; unchanged polling should stay quiet.

Use supported scheduling mechanisms, not a long blocking sleep or a workaround cron.
If the product cannot inspect usage or schedule a wake, say so rather than promising
automatic resumption. Explain any local app/network/awake prerequisites when relevant.
Never redeem reset credits, buy credits or enable paid services without explicit approval.

See [delivery rules](delivery-rules.md) for verification and authorization boundaries.
