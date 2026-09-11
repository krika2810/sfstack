# Playbook: bug-fix

Reproduce, root-cause, fix, verify - in that order, in a real org.

1. **Reproduce first.** Build the failing case in a scratch org: seed the data, run the operation (sf data, anonymous Apex, or UI drive), capture the failure with a debug log. No reproduction, no fix - guesses about the order of execution are how bugs get worse.
2. **Root-cause with the order of execution in view.** Well-Architected warns that the code throwing the limit error is often not the cause: trace what ran earlier in the transaction (recursion, overlapping Flow/trigger work, managed packages). `/sf-how` if the path is unfamiliar.
3. **Failing test** (`/sf-tdd`): the reproduction as an Apex test, failing for the right reason.
4. **Smallest fix** at the root, with bulk-safety and limit budget respected.
5. **Verify in the real org**: test suite green, the reproduction now passes, debug log shows the fixed path, evidence captured (test-run ID, log excerpt).
6. **Blast-radius** (`/sf-blast-radius`) if the fix touches shared metadata or widely-called code.
