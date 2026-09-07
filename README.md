# Customer Support Agent Evaluation — Submission

This folder is the complete, self-contained project submission. It contains the 150-ticket evaluation dataset, the completed analysis workbook, preserved model and judge outputs, an executable verification notebook, LangSmith trace evidence, and the final report.

The files outside this folder in the source repository are instructor-provided starter materials and are not required to review or rerun this submission.

## Main results

- Instructor reference results: 121/150 correct (80.7%), with 29 failures manually annotated and clustered.
- Controlled Nebius baseline: 141/150 correct (94.0%).
- Improved prompt: 149/150 correct (99.3%).
- Target-category result: all 9 `Account_Workflow_Missed` failures were fixed.
- Regression: one ticket (`t050`) changed from correct to incorrect.
- Optional LLM judge: both tested judge models matched the human labels on all 29 reviewed failures.

## What to review

- `report/Customer Support Agent Evaluation Report.docx` — polished final report.
- `workbook/Customer Support Evaluation - Working Copy.xlsx` — completed workbook with annotations, clusters, analysis, and comparisons.
- `notebook/customer_support_evaluation_executed.ipynb` — executable evidence notebook with preserved outputs.
- `results/` — prompts, per-ticket model results, judge results, and operational metrics.
- `langsmith-evidence/` — baseline/improved trace screenshots and trace-review notes.
- `report/evaluation-methodology-note.md` — reusable evaluation workflow learned from the project.

## Verify the submitted results without API keys

1. Install the packages listed in `requirements.txt`.
2. Start Jupyter from this submission folder.
3. Open `notebook/customer_support_evaluation_executed.ipynb`.
4. Leave `RUN_LIVE = False` and `RUN_JUDGE_LIVE = False`.
5. Run all cells. The notebook loads the preserved CSV results, verifies row counts and trace IDs, recomputes accuracy/fixes/regressions, and displays the LangSmith screenshots.

## Create fresh model and LangSmith runs

1. Copy `.env.example` to `.env` and enter your own Nebius and LangSmith API keys.
2. Confirm LangSmith tracing is enabled in `.env`.
3. Set `RUN_LIVE = True` in the notebook to rerun the baseline and improved classifiers over the same 150 tickets.
4. Set `RUN_JUDGE_LIVE = True` to rerun both judges over the 29 human-labeled failures.
5. Run all cells. Fresh CSVs and locally measured latency/token summaries are written to `results/`.

The `.env` file is intentionally excluded because API keys must never be submitted. Live results may vary slightly if the hosted model or provider changes; the preserved CSVs, run IDs, and screenshots are the evidence for the reported experiment.

## Reproducibility design

- Fixed dataset and ticket order.
- Temperature fixed at zero.
- Exact same classifier model for baseline and improved runs.
- Structured output constrained to the five permitted categories.
- Only one account-workflow disambiguation rule was added to the improved prompt.
- Each result row includes its LangSmith run ID.
- Latency and token statistics for the completed runs are preserved in `results/operational_metrics.csv`.

The Loom/Zoom walkthrough link should be supplied in the course submission form alongside this folder or its ZIP archive.
