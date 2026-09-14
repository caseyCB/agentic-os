# Work Log: chore/release-v1.8.27

## Header

- Branch: `chore/release-v1.8.27`
- Classification: `quick-win`
- Classified by: `claude-opus-5`
- Frozen: `2026-09-14`
- Created Date: `2026-09-14`
- Owner: `KbWen`
- Guardrails Mode: `Quick`
- Current Phase: `ship`
- Diff Base SHA: `7bc4955`
- Checkpoint SHA: `7bc4955`
- Recommended Skills: `none`
- Primary Domain Snapshot: `release metadata`
- SSoT Sequence: `171`

---

## Session Info

- Agent: `claude-opus-5`
- Session: `2026-09-14T05:46:46Z`
- Platform: `claude-code`

---

## Task Description

Cut release v1.8.27 on the owner's request after PR #435 merged. Bump the seven canonical version surfaces plus `CITATION.cff` `date-released`, and write the CHANGELOG entry for the four units merged since v1.8.26 (#425, #436, #437/#438, #435). No engine, gate, or configuration change.

---

## Phase Sequence

| Phase | Status | Entered | Notes |
|---|---|---|---|
| bootstrap | done | 2026-09-14 | quick-win on the `chore/release-v1.8.26` precedent; branch from `origin/main` `7bc4955`, upstream tracking removed so a bare push cannot target main |
| plan | done | 2026-09-14 | 8 surfaces + CHANGELOG; adopter delta measured before writing notes |
| implement | done | 2026-09-14 | asserted single-occurrence bumps; CHANGELOG entry |
| review | n/a | - | quick-win: optional; notes checked against each unit's Ship History record |
| test | done | 2026-09-14 | release consistency guard + validators locally; full suite on PR CI |
| handoff | n/a | - | quick-win exempt |
| ship | done | 2026-09-14 | SSoT Ship History (rotated) + heartbeat 172, log archived, INDEX chained |

---

## Phase Summary

- bootstrap/plan: `git diff --name-only v1.8.26..7bc4955` gives 26 files; intersected with `deploy_manifest_golden.txt`, 6 reach an adopter: `deploy.sh`, `repo-gotchas.md`, `trigger-compact-index.json` (core), the `production-readiness` and `systematic-debugging` `SKILL.md` (scaffold), `current_state.md` (scaffold, adopter copy preserved). `security.yml`, `docs/INSTALL.md`, tests and records are upstream-only.
- implement: 7 surfaces + `date-released` bumped by `scratchpad/bump_version.py` (each replace asserts exactly 1 occurrence; zh-TW files re-decoded as UTF-8 after write). CHANGELOG leads with the one action an adopter may need (`git rm -r --cached .agentcortex/tools/__pycache__`), because the ignore rule does not untrack already-committed bytecode and deploy runs no git commands. The #437 bullet says no trigger-rate change was measured, matching that unit's own carried limitation.
- ship: see Final Verification. Post-merge steps are NOT done at merge: lightweight tag `v1.8.27` + `gh release create --latest` (repo-gotchas §12).

⚡ ACX

---

## Gate Evidence

- Gate: bootstrap | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T05:46:46Z
- Gate: plan | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T05:47:00Z
- Gate: implement | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T05:47:44Z
- Gate: test | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T05:48:00Z
- Gate: ship | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T05:48:51Z

---

## External References

| Type | Path / URL | Notes |
|---|---|---|
| PR | https://github.com/KbWen/agentic-os/pull/435 | the downstream fix this release carries |
| Guard | `tests/ci/test_release_version_consistency.py` | pins all 8 surfaces to `deploy.sh` |

---

## Known Risk

- Bytecode already committed by an adopter stays tracked; only the release notes carry the one-line cleanup.
- Release is incomplete at merge: tag + GitHub Release are manual (forgotten twice before). Recorded here and in the PR body before merging.
- Rollback: revert the release commit; delete the tag and Release if already published.

---

## Decisions

none

---

## Conflict Resolution

none

---

## Skill Notes

none

---

## Drift Log

- SSoT write script aborted on its own sequence assertion before any write (a `sed` edit to the copied script had not applied); corrected by hand and re-run. `git status` showed `current_state.md` unmodified between the two runs.
- Ship History rotated at cap 10: `Ship-main-2026-08-27` -> `archive/ship-history-2026.md`, guarded half first.

---

## Review Feedback

none

---

## Red Team Findings

none

---

## Design Reference

none

---

## Observability

none

---

## Resume

none

---

## Test Gate Results

- `pytest tests/ci/test_release_version_consistency.py` -> 2 passed, exit 0 (after the bump and CHANGELOG insert).
- Whole suite: not re-run locally for a version-string cut; it runs on the PR's CI (Linux + 3 Windows shards) before merge. The code it would exercise is unchanged since PR #435's local full run and green CI.

---

## Evidence

- Stale-version sweep after bump: `git grep 1.8.26` outside `CHANGELOG.md`, `archive/`, `current_state.md` and the backlog -> no hits.
- zh-TW banner diffs byte-identical except the version digits (`cat -v` of both sides).

## Final Verification

> Sole location of this cut's closing figures, taken against `386d522` (tree clean) after every other write.

- `validate.sh` exit 0 and `validate.ps1` exit 0: both `pass=99 warn=4 fail=0 skip=3`, identical; all 4 WARNs pre-existing
- `test_release_version_consistency.py` 2 passed; `check_audit_chain.py` intact; `check_ssot_caps.py` ship history 10/10

⚡ ACX
