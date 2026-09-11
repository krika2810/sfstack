# Playbook: Opening a PR

The last step of every other playbook. The PR is how the work gets reviewed, so it carries the evidence the playbooks produced.

## Steps

1. **Order the commits.** Small, in the order a reviewer wants to read them: failing test before fix (bug-fix), schema before logic (feature), no commit that breaks the suite. Squash the noise: "wip" and "fix typo" commits get amended away.
2. **Write the body as a briefing.** Plain English, in this order:
   - What changed and why, in two or three sentences.
   - The evidence: test-run IDs, deployment IDs, validation job IDs, before/after numbers, screenshots. Paste the key excerpts, do not describe them.
   - The decisions a reviewer must know: access choices (`with sharing`, `stripInaccessible`), async boundaries, anything deferred.
3. **Self-review the diff first.** Read the whole diff as if you were the reviewer. Apply **principle-minimize-reader-load** and **sf-unslop** to the code and the prose. Remove debug statements, commented-out code, and drive-by changes that belong in a different PR.
4. **Push and open.** Push the branch and open the PR with the body from step 2. Attach the evidence files if they are large (decision trail TSV, log excerpts) rather than inlining them.
5. **Report the link and the IDs.** The PR URL and the headline evidence in the reply.

## Gotchas and failure modes

- **PRs that mix concerns.** A bug fix and a refactor in one PR makes both unreviewable and unrevertable. Split them.
- **Evidence-free bodies.** "Tests pass" is a claim. The test-run ID and output are evidence. Reviewers trust rerunnable proof.
- **The diff that grew.** If the diff is bigger than the playbook expected, say why in the body or split it. Do not let a reviewer discover it.

## Proof it worked

The PR exists with the ordered commits and the evidence in the body, and the CI or validation evidence is linked.

## Reply

PR link, headline evidence, and anything you deliberately left out of scope.
