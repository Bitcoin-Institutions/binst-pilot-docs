# Use Cases

The pilot's architecture — institution + process template + process
instance — is generic. Any multi-step workflow that benefits from
on-chain auditability and Bitcoin-anchored identity can be modeled.

---

## What the pilot demonstrates

The `demo-flow.ts` script runs a complete lifecycle:

1. Deploy an institution with a named admin
2. Bind a Bitcoin inscription ID to the institution
3. Add members to the institution
4. Create a process template with defined steps
5. Instantiate the process
6. Execute each step sequentially, recording actor and timestamp
7. Complete the process instance

This generic flow maps to any domain where **who did what, when, and
in what order** matters.

## Example mappings

| Domain | Institution | Process Template | Steps |
|---|---|---|---|
| Public admin | Municipal office | Permit application | Submit → Review → Approve/Reject |
| Private sector | Company HR | Hiring workflow | Post → Screen → Interview → Offer |
| Legal | Arbitration body | Dispute resolution | File → Assign mediator → Hear → Decide |
| Governance | Community org | Proposal lifecycle | Draft → Discuss → Vote → Execute |
| Supply chain | Manufacturer | Quality inspection | Sample → Test → Report → Certify |

Each row is a different parameterization of the same four contracts.
The pilot doesn't implement domain-specific logic — it provides the
framework that any of these domains can plug into.
