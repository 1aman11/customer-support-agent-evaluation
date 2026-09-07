# Customer Support Agent Evaluation

## Project goal and context

This project evaluates an e-commerce customer-support routing agent that assigns each incoming ticket to one of five operational queues: `order_status`, `refund_request`, `product_issue`, `account_help`, or `other`. Accurate routing matters because a wrong classification sends the customer to the wrong team, creating avoidable transfers, slower resolution, and additional support effort.

Our goal was not only to improve this classifier, but to learn and demonstrate a repeatable agent-evaluation process that can be applied to future AI systems. Using a fixed 150-ticket labeled dataset, we reviewed the ground truth, established a reproducible baseline, annotated and clustered recurring failures, selected one high-impact category, made one focused prompt change, and reran the same cases to measure improvements and regressions. LangSmith traces connected aggregate metrics to individual decisions, while an LLM judge was calibrated against human labels. This report documents both the measured result and the evaluation workflow that produced it.

## Evaluation one-liner

I measured exact-match routing accuracy, account-workflow failure count, regression count, latency, and token usage for an e-commerce support classifier on a fixed 150-ticket labeled dataset, using code-based comparison against reviewed ground-truth categories and LangSmith-traced baseline and improved runs. The pass bar was at least 95% accuracy, zero remaining failures in the selected target category, no more than one regression, and p95 latency below 1.5 seconds.

## Agent and user outcome

The agent classifies each customer ticket into exactly one of five queues: `order_status`, `refund_request`, `product_issue`, `account_help`, or `other`. The user outcome is accurate routing to the team that can resolve the customer's primary problem without avoidable transfers or delays.

## Dataset and ground truth

The fixed evaluation dataset contains 150 instructor-provided tickets: 24 `order_status`, 42 `refund_request`, 24 `product_issue`, 30 `account_help`, and 30 `other`. It includes direct requests, ambiguous multi-intent tickets, misleading surface keywords, post-purchase adjustments, checkout and account-access failures, and general inquiries.

The supplied labels were treated as the assignment rubric and reviewed during failure annotation. Some refund-versus-underlying-problem cases require subjective tie-breaking, so they were retained but identified as higher-noise examples. The `Account_Workflow_Missed` cases provided a more internally consistent target for a controlled prompt experiment.

## Metrics and judging method

| Metric | Method | Pass bar |
|---|---|---:|
| Overall routing accuracy | Exact match against the supplied true category | At least 95% |
| Account-workflow failures | Exact match plus human-reviewed failure grouping | 0 after improvement |
| Regressions | Per-ticket baseline/improved comparison | No more than 1 |
| Latency | LangSmith trace duration | p95 below 1.5 seconds |
| Token usage | LangSmith prompt and completion token totals | Report and monitor as a cost proxy |

Exact match is the primary classifier evaluator because every ticket has one deterministic category label; human review was used to explain and cluster failures. The optional LLM-as-a-judge evaluation was also completed as a separate binary task: determine whether each of the 29 reference failures belongs to `Account_Workflow_Missed`.

## Instrumentation

Tracing was enabled in the LangSmith project `customer-support-evals`. Each ticket has one parent trace whose inputs, outputs, and metadata expose the ticket ID, run name, prompt version, true category, prediction, and reasoning; correctness is recorded in the paired result row by comparing the true and predicted categories. Child spans record prompt construction, the Nebius model call, the runnable sequence, and structured-output parsing. The baseline and improved runs use the same trace structure so individual cases can be compared directly.

## Instructor reference analysis

The instructor-provided workbook contains 121 correct predictions out of 150, or 80.7% accuracy, with 29 failures. Those failures were annotated and grouped into:

| Failure category | Count |
|---|---:|
| `Refund_Intent_Missed` | 12 |
| `Account_Workflow_Missed` | 9 |
| `Scope_Boundary_Misread` | 8 |

This reference run was used for qualitative failure discovery. It was not used as the numerical baseline for the prompt comparison because its model configuration differs from our Nebius run.

## Reproducible baseline

The baseline used the unchanged instructor classifier prompt with Nebius `openai/gpt-oss-120b`, temperature 0, the fixed 150 tickets, and structured output.

| Measure | Baseline result |
|---|---:|
| Correct predictions | 141/150 |
| Accuracy | 94.0% |
| p50 latency | 0.603 seconds |
| p95 latency | 0.936 seconds |
| Total tokens | 45,145 |
| Average tokens per ticket | 301.0 |

All nine baseline failures were true `account_help` tickets. Three app/order-history access cases were routed to `other`, three rejected promo-code cases were routed to `other`, and three missing account-history order records were routed to `order_status`.

Representative baseline failure: `t105`, trace `01a07862-7f92-7622-93a9-f042272b577e`. The model correctly recognized an app crash but concluded that the technical issue was not covered by the listed categories, showing a gap in the original `account_help` definition.

## Improvement hypothesis and prompt change

`Account_Workflow_Missed` was selected because these errors can block checkout or account access and the labels form a consistent, measurable cluster. Although `Refund_Intent_Missed` had more cases in the instructor reference run, several depend on less consistent tie-breaking between the underlying problem and the requested remedy.

One focused disambiguation rule was added:

> If the customer cannot complete an account-linked action because of repeated logout, an app or site crash, a rejected promo code at checkout, or a missing order record in account history, classify as account_help. Do not route only from surface words such as order, product, or discount.

No category definitions, model settings, dataset rows, evaluator logic, or output schema were otherwise changed.

## Post-improvement results

| Measure | Baseline | Improved | Delta |
|---|---:|---:|---:|
| Correct predictions | 141/150 | 149/150 | +8 |
| Accuracy | 94.0% | 99.3% | +5.3 percentage points |
| Account-workflow failures | 9 | 0 | -9 |
| Regressions | 0 | 1 | +1 |
| p50 latency | 0.603 s | 0.574 s | -0.029 s |
| p95 latency | 0.936 s | 0.997 s | +0.061 s |
| Total tokens | 45,145 | 56,161 | +11,016 |
| Average tokens per ticket | 301.0 | 374.4 | +73.4 |

The revised prompt met the quality, target-failure, regression, and latency pass bars. It fixed all nine targeted failures: `t105`–`t107` and `t114`–`t119`. The prompt increased token usage by 24.4%, which is the principal operational tradeoff.

Representative fixed traces:

- `t105`, improved trace `01a07895-78b9-7fb3-ba18-415f5867ff76`: app crash while opening order history changed from `other` to `account_help`.
- `t114`, baseline trace `01a07862-95f5-71b0-942c-56c3228f18d8`; improved trace `01a07895-90cf-7651-99c3-1c47ddcfa216`: rejected promo code changed from `other` to `account_help`.
- `t117`, baseline trace `01a07862-9e57-7821-ab3f-e03222f04215`; improved trace `01a07895-98e2-7f62-94b4-147c5a6e062b`: missing account-history record changed from `order_status` to `account_help`.

## Optional LLM-as-a-judge evaluation

The judge received only the customer message and the classifier's incorrect predicted intent. It returned TRUE when the error matched `Account_Workflow_Missed` and FALSE for the other failure categories. The human Step 3 labels supplied the binary ground truth: 9 TRUE cases and 20 FALSE cases.

| Judge model | Agreement | TRUE precision/recall/F1 | FALSE precision/recall/F1 | p50 latency | p95 latency | Total tokens |
|---|---:|---:|---:|---:|---:|---:|
| `openai/gpt-oss-120b` | 100.0% (29/29) | 1.00 / 1.00 / 1.00 | 1.00 / 1.00 / 1.00 | 0.772 s | 1.119 s | 13,945 |
| `Qwen/Qwen3-235B-A22B-Instruct-2507` | 100.0% (29/29) | 1.00 / 1.00 / 1.00 | 1.00 / 1.00 / 1.00 | 0.927 s | 1.283 s | 10,046 |

There were no mismatches, so prompt calibration was not warranted. `openai/gpt-oss-120b` was selected because it tied on agreement, ran faster, and produced more concise explanations. Qwen used fewer tokens but was slower and more verbose.

Representative judge traces:

- TRUE case `t103`, selected-judge trace `01a078c7-1d3a-70f2-ac57-b1c7f28c2a9d`: repeated logout during payment with a non-`account_help` prediction satisfied both rubric conditions.
- FALSE case `t024`, selected-judge trace `01a078c6-ff5c-7241-bef3-4b8f718cbe2d`: a wrong delivery address was correctly treated as a delivery problem rather than an account workflow.

## Regression and next evaluation cycle

The only improved-run failure was `t050`: “Item went on sale right after I ordered. Do you do price adjustments?” It changed from the correct `refund_request` prediction to `other`. Improved trace: `01a07894-e2e7-7052-9702-6df416728f61`.

The next experiment should add a separate, focused rule stating that post-purchase price adjustments belong to `refund_request`. That change should be evaluated as a new prompt version against the same 150-ticket dataset, with particular regression checks for pre-purchase discount questions and promo-code failures. It was not added to the current prompt because doing so would introduce a second intervention and weaken attribution of the measured result.

## Production monitoring

Retain prompt version, model, run name, ticket ID, expected category when available, predicted category, correctness, latency, token usage, and error status in LangSmith. Monitor a rolling labeled sample for accuracy below 95%, any reappearance of account-workflow failures, regression growth above 1%, p95 latency above 1.5 seconds, and token usage more than 25% above the current improved baseline.
