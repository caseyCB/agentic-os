# Work Log: fix/downstream-ignore-python-bytecode

## Header

- Branch: `fix/downstream-ignore-python-bytecode`
- Classification: `quick-win`
- Classified by: `claude-opus-5`
- Frozen: `2026-09-14`
- Created Date: `2026-09-14`
- Owner: `KbWen`
- Guardrails Mode: `Quick`
- Current Phase: `ship`
- Diff Base SHA: `3667fe6`
- Checkpoint SHA: `52ef638`
- Recommended Skills: `none`
- Primary Domain Snapshot: `deploy`
- SSoT Sequence: `170`

---

## Session Info

- Agent: `claude-opus-5`
- Session: `2026-09-14T02:04:28Z`
- Platform: `claude-code`

---

## Task Description

Take over contributor PR #435 (issue #430, backlog #191) after a week without response to the requested change, with the user's approval. The submitted fix leaves the deployed `.gitignore` growing by 14 lines on every re-deploy; finish it so the framework's bytecode is ignored without cluttering, or changing the ignore policy of, the adopter's own tree.

---

## Phase Sequence

| Phase | Status | Entered | Notes |
|---|---|---|---|
| bootstrap | done | 2026-09-14 | quick-win; SSoT read; contributor branch checked out, `origin/main` merged in (no force-push) |
| plan | done | 2026-09-14 | 2 product files + backlog row; see Phase Summary |
| implement | done | 2026-09-14 | `ceed4c5`; review fixes `6a3de6e` |
| review | done | 2026-09-14 | fresh same-vendor subagent, diff + issue + owner constraint only; 0 blockers, 2 should-fix (both fixed), 3 nits |
| test | done | 2026-09-14 | full CI-equivalent suite at `6a3de6e` (1 own failure, fixed `52ef638`) + changed-file rerun |
| handoff | skipped | - | quick-win exempt per AGENTS.md §Delivery Gates |
| ship | done | 2026-09-14 | SSoT Ship History (rotated at cap 10) + heartbeat 171, backlog #191 Shipped, log archived, INDEX chained |

---

## Phase Summary

- bootstrap: quick-win (2 modules: `deploy.sh` ignore block + its test; no gate/engine change). Reproduced the maintainer's review finding on a real deploy before planning, see Evidence.
- plan: (1) `deploy.sh`: replace the two repo-wide lines with one framework-scoped line `.agentcortex/**/__pycache__/`, and add it to the `managed[]` strip table. (2) `test_deploy_tiering.py`: replace the whole-file grep test with one that parses the heredoc block and the `managed[]` table separately and asserts every block entry is strippable, plus that every `.py` in the deploy golden lives under `.agentcortex/` so the scoped line covers it; add one behavioral test that deploys twice into a real git repo with adopter ignore rules and asserts byte-identical output, adopter lines preserved, framework bytecode ignored, adopter bytecode NOT ignored. (3) backlog #191 `Pending -> In Progress` now, `Shipped` at ship. Rollback: `git revert` the implement commit. Not touched: `deploy.ps1` (delegates to `deploy.sh`, `deploy.ps1:76`), validators (required-pattern list, not exhaustive), CHANGELOG (written at release cut).
- implement: as planned (`ceed4c5`), plus a one-line section comment instead of two to keep the adopter's file short. All three tests mutation-verified. | Confidence: 92% - high
- review: fresh reviewer found no blocker. Adjudicated each claim by reproducing it rather than accepting it: (R1) test outcome depended on the host's global git excludes, reproduced -> fixed with an empty `core.excludesFile`; (R2) mixed-version deploys grow `.gitignore` because an older `deploy.sh` stops stripping at an unknown entry, reproduced +10..13 lines/round -> entry moved last (D-2), now +3/round and 1 stray line on a single downgrade; (N1) no untrack hint in the deploy banner -> closed, would print for every adopter, cleanup goes in release notes; (N2) backlog row #191 text prescribes the repo-wide pair -> rewrite at ship; (N3) PR thread needs the takeover explanation -> planned comment.
- test: full suite caught a 3.9-floor violation in the new test that no targeted run had included; fixed in `52ef638`; see Drift Log and Test Gate Results.
- ship: PASS on `52ef638`; SSoT sequence 170 -> 171, Ship History rotated (`Ship-fix-ignore-assertion-binding-2026-08-24` -> `archive/ship-history-2026.md`); archived to `archive/fix-downstream-ignore-python-bytecode-20260914.md`. Quick-win knowledge nudge: no L2 line, the non-obvious constraints are encoded where the next editor meets them (the `managed[]` comment in `deploy.sh` and the namespace test's assertion messages).

⚡ ACX

---

## Gate Evidence

- Gate: bootstrap | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T02:04:28Z
- Gate: plan | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T02:05:10Z
- Gate: implement | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T02:22:51Z
- Gate: review | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T02:46:46Z
- Gate: test | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T04:11:07Z
- Gate: ship | Verdict: PASS | Classification: quick-win | Timestamp: 2026-09-14T04:11:55Z

---

## External References

| Type | Path / URL | Notes |
|---|---|---|
| Spec | - | - |
| ADR | docs/adr/ADR-005-downstream-file-preservation-tiering.md | framework manages only its own downstream namespace |
| Issue | https://github.com/KbWen/agentic-os/issues/430 | backlog #191 |
| PR | https://github.com/KbWen/agentic-os/pull/435 | contributor PR, taken over in place (maintainerCanModify) |

---

## Known Risk

- A future framework `.py` shipped outside `.agentcortex/` would not be covered by the scoped line. Mitigation: the planned golden-manifest test fails in that case.
- Adopters who already committed `.agentcortex/tools/__pycache__/` keep it tracked; `.gitignore` does not untrack. Deploy will not run `git rm` in an adopter repo; the release notes should carry the one-line cleanup.
- Mixed framework versions deploying into one repo still add 3 `.gitignore` lines per old->new round; a single downgrade leaves 1 stray line. Caused by older `deploy.sh` code; same class already applied to every earlier entry addition. The "add new entries last" rule is a comment, not a check.

---

## Decisions

### D-1: framework-scoped pattern instead of repo-wide `__pycache__/` + `*.pyc`

- **Decision**: ship `.agentcortex/**/__pycache__/` only.
- **Reason**: the repo-wide pair also ignores the adopter's own bytecode anywhere in their tree (measured: `app/legacy.pyc` ignored), a silent policy change caused by installing the framework, and duplicates the rule most Python adopters already have. All 19 deployed `.py` files live in `.agentcortex/tools/`; Python 3 writes bytecode only into `__pycache__/` (PEP 3147), so `*.pyc` adds nothing for framework files.
- **Alternatives**: the submitted two lines plus `managed[]` entries (the reviewer's minimum; fixes growth but keeps the policy change).

→ local

### D-2: new block entries go last; no marker-bounded strip in this change

- **Decision**: emit the entry as the block's last line and say so beside `managed[]`.
- **Reason**: already-deployed `deploy.sh` versions cannot be changed; placement is the only lever over what they leave behind (measured 13 -> 3 lines per alternation round).
- **Alternatives**: strip everything between the markers regardless of the table (reviewer's follow-up). Rejected here: it only helps future downgrades to this version, and it silently deletes lines an adopter added inside the block, which today survive; it also needs a separate rule for legacy blocks with no end marker. Cleaning stray lines above the marker was also rejected: they cannot be told apart from adopter-written lines.

→ local

---

## Conflict Resolution

none

---

## Skill Notes

none

---

## Drift Log

- Full CI-equivalent suite at `6a3de6e` (951 collected, local 3.14.3, 69m33s, run alone): `1 failed, 949 passed, 1 skipped`, exit 1. Failure was mine: `test_write_text_newline_ratchet` flagged `test_deploy_tiering.py:480 [write_text(newline=) is 3.10+]` in the new double-deploy test (repo floor is 3.9; the ratchet exists for exactly this). Fixed with `write_bytes`. The 3 targeted runs and the mutation runs had not included the ratchet file, which is why only the full suite saw it.
- An earlier full-suite run at `ceed4c5` was stopped at ~15% (contention with the scenario battery and the reviewer, and superseded by review fixes); no result from it is claimed.
- Scope: the reviewer's comment asked only for two `managed[]` lines; D-1 narrows the patterns themselves after the user asked that downstream not be cluttered or given extra trouble. Contributor's commit is kept; changes land as new commits on top.
- Recovered stale Work Log lock on 2026-09-14T04:11:28.482547+00:00; prior_owner=KbWen; prior_session=claude-opus-5-2026-09-14T02:04:28Z; reason=stale-time; lock=fix-downstream-ignore-python-bytecode.lock.json

---

## Review Feedback

none

---

## Red Team Findings

- R1 (should-fix, fixed `6a3de6e`): host global excludes decided the adopter-policy assertion. Repro: `GIT_CONFIG_GLOBAL` with `excludesFile` = `__pycache__/` -> `AssertionError: the adopter's own bytecode must be left...`; after fix 3 passed under the same config.
- R2 (should-fix, mitigated `6a3de6e`, residual in Known Risk): mixed-version growth. Repro `scratchpad/mixed_versions.sh`: mid-block `new:36 old:46 old:46 new:49 new:49 old:59 new:62 old:72`; last-in-block `new:36 old:36 old:36 new:39 new:39 old:39 new:42 old:42`.

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

- `python -m pytest tests/ci/ tests/guard/ .agentcortex/tests/ -q` at `6a3de6e`, 951 collected per `--collect-only`, run alone: `1 failed, 949 passed, 1 skipped in 4173.64s`, exit 1 -> `test_write_text_newline_ratchet::test_no_write_text_newline_calls` (own defect).
- After fix `52ef638`: `pytest tests/ci/test_write_text_newline_ratchet.py tests/ci/test_deploy_tiering.py -q` -> `43 passed, 1 skipped in 516.45s`, exit 0. Changed-file rerun, not a second whole-suite run.

---

## Evidence

- Baseline, PR as submitted at `3667fe6`, 3 real `deploy.sh` runs into fresh git repos (`scratchpad/ignore_sim.sh`): fresh target `.gitignore` lines 33 -> 47 -> 61; Python-adopter target 39 -> 53 -> 67; `__pycache__/` occurs 3x / 4x; managed lines leak outside the block. Adopter `app/legacy.pyc` ignored: yes (policy change).
- Control `origin/main` 924a2dd, same script: 29/29/29 and 35/35/35 lines (idempotent), framework `.pyc` visible to `git status` after `validate.sh`: 2 (the #430 bug).
- `ceed4c5`, same script: 33/33/33 and 39/39/39, adopter lines kept, visible `.pyc` 0, adopter's own `app/legacy.pyc` not ignored.
- Mutation (`ceed4c5`, restored byte-identical via `cmp`): drop `managed[]` entry -> strippable + double-deploy FAIL; repo-wide pair with `managed[]` entries -> namespace + double-deploy FAIL; golden gains `.agents/skills/demo/run.py` -> namespace FAIL. Re-run on `6a3de6e` under a polluted global excludes file: same FAILs for the first two; clean tree 3 passed.
- `pytest tests/ci/test_deploy_tiering.py` at `ceed4c5` -> 41 passed, 1 skipped, exit 0 (read via PIPESTATUS).
- Demonstration, `deploy.ps1` vs `deploy.sh` into fresh targets (ps1 deployed twice): `.gitignore` sha `FFFF383F07` both, pattern present once.
- Scripts named `scratchpad/*.sh` are session-local and not committed; each line records its command shape and result.
- Re-run at `6a3de6e` (entry moved last): battery results identical to the `ceed4c5` line below except `.gitignore` shas; `deploy.ps1` vs `deploy.sh` sha `C52B22DE30` both, block ends with the bytecode entry.
- Downstream battery at `ceed4c5` (`scratchpad/downstream_scenarios.sh`): upgrade from main 33 -> 36 -> 36, validate `pass=86 warn=1 fail=0 skip=8` before and after; committed `.pyc` stay tracked until `git rm -r --cached`, then 0/0; legacy `AI Brain OS` block replaced, idempotent; CRLF adopter file idempotent on main and branch; subdirectory install ignores only its own bytecode, root `.gitignore` untouched; `--no-python` summary identical to main, 0 `.pyc`.

## Final Verification

> Sole location of this unit's closing figures, taken against `a45f4f6` (tree clean) after every other write.

- `validate.sh` exit 0 and `validate.ps1` exit 0: both `pass=99 warn=4 fail=0 skip=3`, identical; all 4 WARNs pre-existing (none name this log)
- 19 SSoT/backlog/chain-sensitive test files + the 3.9 ratchet -> `388 passed`, exit 0
- `check_audit_chain.py` intact; `check_ssot_caps.py` ship history 10/10

⚡ ACX
