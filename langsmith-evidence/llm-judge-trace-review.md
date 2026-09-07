# LLM-as-a-judge trace review

The binary judge evaluated all 29 instructor-reference classifier failures against the human-labeled `Account_Workflow_Missed` category. The judge saw the customer message and the classifier's predicted intent, but not the hidden failure-category label.

## Model comparison

| Model | Agreement | p50 latency | p95 latency | Total tokens |
|---|---:|---:|---:|---:|
| `openai/gpt-oss-120b` | 100.0% (29/29) | 0.772 s | 1.119 s | 13,945 |
| `Qwen/Qwen3-235B-A22B-Instruct-2507` | 100.0% (29/29) | 0.927 s | 1.283 s | 10,046 |

Both models correctly detected all nine positives and rejected all 20 negatives. `openai/gpt-oss-120b` was selected because it was faster and produced more concise explanations. Qwen used fewer tokens but was slower and more verbose.

## Representative TRUE trace

- Ticket: `t103`
- Input: Site logs me out every time I try to pay.
- Classifier prediction: `product_issue`
- Human label: TRUE
- Judge prediction: TRUE
- Selected-judge trace: `01a078c7-1d3a-70f2-ac57-b1c7f28c2a9d`
- Review: The judge cited both required conditions: repeated logout during payment is a qualifying account-linked workflow, and the predicted intent was not `account_help`.

## Representative FALSE trace

- Ticket: `t024`
- Input: The carrier marked it delivered but it shows the wrong delivery address.
- Classifier prediction: `account_help`
- Human label: FALSE
- Judge prediction: FALSE
- Selected-judge trace: `01a078c6-ff5c-7241-bef3-4b8f718cbe2d`
- Review: The judge correctly applied the exclusion rule for delivery/address problems instead of treating the word “address” as proof of an account workflow.

No mismatches were available for prompt calibration. Future validation should add new, less template-aligned cases to test whether the perfect agreement generalizes beyond this fixed dataset.
