<!-- markdownlint-disable -->

# Hardening Report: hyperdevs-team--check-new-commits-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **hyperdevs-team--check-new-commits-action/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable version tags rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the upstream tag is moved or compromised.

In `.github/workflows/ci.yml`:
- `uses: actions/checkout@v6` (line 25)
- `uses: actions/setup-node@v6` (line 30, line 47)

In `.github/workflows/release-action.yml`:
- `uses: actions/checkout@v6` (line 16)
- `uses: actions/setup-node@v6` (line 22)
- `uses: softprops/action-gh-release@v3` (line 47)

All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/ci.yml:25`
- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:47`
- `.github/workflows/release-action.yml:16`
- `.github/workflows/release-action.yml:22`
- `.github/workflows/release-action.yml:47`

### script-injection (severity: high)

Sub-rule (a): The 'Print output' step in ci.yml directly interpolates `${{ steps.test-action.outputs.has-new-commits }}` and `${{ steps.test-action.outputs.new-commits-number }}` inside a `run:` shell command string. `steps.*.outputs.*` values are workflow-controllable and flow through YAML template substitution before the shell processes them, enabling script injection if the action ever produces attacker-influenced output. The offending lines are:

  `echo "has-new-commits=${{ steps.test-action.outputs.has-new-commits }}"`
  `echo "new-commits-number=${{ steps.test-action.outputs.new-commits-number }}"`

Fix: move the values into `env:` variables and reference them as quoted shell variables, e.g.:
```yaml
env:
  HAS_NEW_COMMITS: ${{ steps.test-action.outputs.has-new-commits }}
  NEW_COMMITS_NUMBER: ${{ steps.test-action.outputs.new-commits-number }}
run: |
  echo "has-new-commits=$HAS_NEW_COMMITS"
  echo "new-commits-number=$NEW_COMMITS_NUMBER"
```

Locations:

- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned action references by resolving their full commit SHAs via lookup_action_sha: actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38, softprops/action-gh-release@v3 → 3d0d9888cb7fd7b750713d6e236d1fcb99157228. Fixed script injection in ci.yml 'Print output' step by moving ${{ steps.test-action.outputs.has-new-commits }} and ${{ steps.test-action.outputs.new-commits-number }} into an env: block as HAS_NEW_COMMITS and NEW_COMMITS_NUMBER, then referencing them as plain shell variables in the run: script.

