# Template Placeholder Reference

Every placeholder present in a selected template is required. Replace it with
literal text before hashing or distributing the prompt. Paths must be absolute
in generated prompts, hashes must be full lowercase SHA-256 values, and a
missing concept must be written explicitly as `none`, not left unresolved.

## Notebook and drift

| Placeholder | Purpose | Example |
|---|---|---|
| `{{NOTEBOOK_LABEL}}` | Human-readable course and lesson label | `CA09` |
| `{{NOTEBOOK_PATH}}` | Absolute live notebook path | `/path/to/repo/Week_2/09_Lesson.ipynb` |
| `{{NOTEBOOK_SHA256}}` | Expected live notebook hash | `64 lowercase hex characters` |
| `{{APPLIED_SHA256}}` | Expected post-apply live notebook hash | `64 lowercase hex characters` |
| `{{FROZEN_PATH}}` | Absolute frozen-baseline path | `/private/tmp/ca09_frozen.ipynb` |
| `{{FROZEN_SHA256}}` | Frozen-baseline hash | `64 lowercase hex characters` |
| `{{EXACT_PERMITTED_DRIFT_OR_NONE}}` | Complete permitted-drift rule | `none` |

## Candidate and decision artifacts

| Placeholder | Purpose | Example |
|---|---|---|
| `{{CANDIDATE_PATH}}` | Exact candidate under review | `/private/tmp/ca09_candidate.ipynb` |
| `{{CANDIDATE_SHA256}}` | Exact candidate hash | `64 lowercase hex characters` |
| `{{EDITOR_CANDIDATE_PATH}}` | Pre-reconciliation editor candidate | `/private/tmp/ca09_editor_candidate.ipynb` |
| `{{EDITOR_CANDIDATE_SHA256}}` | Editor-candidate hash | `64 lowercase hex characters` |
| `{{EDITOR_REPORT}}` | Editor disposition path | `/private/tmp/ca09_editor_disposition.md` |
| `{{EDITOR_REPORT_SHA256}}` | Editor disposition hash | `64 lowercase hex characters` |
| `{{ADJUDICATION}}` | Reconciliation artifact path when that short name is used | `/private/tmp/ca09_reconciliation.md` |
| `{{ADJUDICATION_PATH}}` | Reconciliation or final adjudication path | `/private/tmp/ca09_reconciliation.md` |
| `{{ADJUDICATION_SHA256}}` | Reconciliation or adjudication hash | `64 lowercase hex characters` |
| `{{DELTA_REPORT}}` | Applied-delta report path | `/private/tmp/ca09_applied_delta.md` |
| `{{DELTA_REPORT_SHA256}}` | Applied-delta report hash | `64 lowercase hex characters` |
| `{{EXACT_APPLIED_SCOPE}}` | Complete approved applied delta | `cells 7 and 12 only, source changes shown below` |

## Review reports

| Placeholder | Purpose | Example |
|---|---|---|
| `{{CAMERA_AUDIT}}` | Camera/meaning audit path | `/path/to/reviewer1.txt` |
| `{{CAMERA_AUDIT_SHA256}}` | Camera/meaning audit hash | `64 lowercase hex characters` |
| `{{TECHNICAL_AUDIT}}` | Technical/proportionality audit path | `/path/to/reviewer2.txt` |
| `{{TECHNICAL_AUDIT_SHA256}}` | Technical/proportionality audit hash | `64 lowercase hex characters` |
| `{{REVIEWER_1_AUDIT}}` | Reviewer 1 code-audit path | `/path/to/reviewer1.txt` |
| `{{REVIEWER_1_AUDIT_SHA256}}` | Reviewer 1 code-audit hash | `64 lowercase hex characters` |
| `{{REVIEWER_2_AUDIT}}` | Reviewer 2 code-audit path | `/path/to/reviewer2.txt` |
| `{{REVIEWER_2_AUDIT_SHA256}}` | Reviewer 2 code-audit hash | `64 lowercase hex characters` |

## Verification, ownership, and design artifacts

| Placeholder | Purpose | Example |
|---|---|---|
| `{{PREFLIGHT_REPORT}}` | Deterministic preflight report path | `/private/tmp/ca09_preflight.md` |
| `{{PREFLIGHT_REPORT_SHA256}}` | Preflight report hash | `64 lowercase hex characters` |
| `{{VERIFY_REPORT}}` | Candidate or applied verification report path | `/private/tmp/ca09_verify.md` |
| `{{VERIFY_REPORT_SHA256}}` | Verification report hash | `64 lowercase hex characters` |
| `{{OWNERSHIP_REPORT}}` | Deleted-claim and concept-ownership report path | `/private/tmp/ca09_ownership.md` |
| `{{OWNERSHIP_REPORT_SHA256}}` | Ownership report hash | `64 lowercase hex characters` |
| `{{DESIGN_BRIEF}}` | Design-gate brief path | `/private/tmp/ca09_design_gate.md` |
| `{{DESIGN_BRIEF_SHA256}}` | Design-gate brief hash | `64 lowercase hex characters` |
| `{{DEMONSTRATED_PROBLEM_AND_PASSING_CONSTRAINTS}}` | Exact problem plus all constraints a design must satisfy | `The baseline misgrades tool narration...` |
| `{{BASELINE_AND_AT_MOST_TWO_DESIGNS}}` | Baseline and no more than two complete design sketches | `Baseline..., Design A..., Design B...` |

## Renders and crops

| Placeholder | Purpose | Example |
|---|---|---|
| `{{BASELINE_RENDER}}` | Complete baseline render path | `/private/tmp/ca09_baseline.png` |
| `{{BASELINE_RENDER_SHA256}}` | Baseline render hash | `64 lowercase hex characters` |
| `{{CANDIDATE_RENDER}}` | Complete candidate render path | `/private/tmp/ca09_candidate.png` |
| `{{CANDIDATE_RENDER_SHA256}}` | Candidate render hash | `64 lowercase hex characters` |
| `{{FINAL_RENDER}}` | Complete post-apply render path | `/private/tmp/ca09_final.png` |
| `{{FINAL_RENDER_SHA256}}` | Final render hash | `64 lowercase hex characters` |
| `{{CROP_MANIFEST}}` | Manifest of full adjacent-run crops | `/private/tmp/ca09_crops.txt` |
| `{{CROP_MANIFEST_SHA256}}` | Crop-manifest hash | `64 lowercase hex characters` |

## Governing artifacts

| Placeholder | Purpose | Example |
|---|---|---|
| `{{GUIDELINES_PATH}}` | Absolute local design-guidelines path | `/path/to/repo/design_guidelines.md` |
| `{{GUIDELINES_SHA256}}` | Design-guidelines hash | `64 lowercase hex characters` |
| `{{OUTLINE_PATH}}` | Absolute course-outline path | `/path/to/repo/course_outline.md` |
| `{{OUTLINE_SHA256}}` | Course-outline hash | `64 lowercase hex characters` |
| `{{PROFILE_PATH}}` | Course review profile path | `/path/to/repo/.claude/course_review_profile.md` |
| `{{PROFILE_SHA256}}` | Course profile hash | `64 lowercase hex characters` |
| `{{CODE_PROCESS_PATH}}` | Code-readability process path | `/path/to/repo/.claude/code_readability_process.md` |
| `{{CODE_PROCESS_SHA256}}` | Code-readability process hash | `64 lowercase hex characters` |

## Reviewer instructions and focused scope

| Placeholder | Purpose | Example |
|---|---|---|
| `{{AUDITOR_LANE}}` | Reviewer-specific independent lane | `source-order teaching and camera meaning` |
| `{{READ_ORDER_INSTRUCTION}}` | Exact baseline/candidate read order | `Read the candidate first, then the baseline.` |
| `{{ALTERNATIVE_ALLOWANCE}}` | Whether and when a different form may be proposed | `only when both pinned forms fail the same contract` |
| `{{EXACT_TRIGGER_AND_SCOPE}}` | Why a focused challenge fired and what it may reopen | `Third implementation in cell 12; cell 12 only.` |
| `{{FOCUSED_TARGETS_WITH_EXACT_SOURCE_AND_CROPS}}` | Prose targets with all compared source and full-run crops | `Target 1: cell 12...` |
| `{{FOCUSED_TARGETS_WITH_EXACT_SOURCE_AND_EVIDENCE}}` | Code targets with exact source and falsifying evidence | `Target 1: cell 22 output boundary...` |

## Completeness check

Before pinning a prompt, run a literal placeholder search over that generated
file. Any remaining double-brace token is a blocking error. When adding or
renaming a template placeholder, update this reference in the same change.
