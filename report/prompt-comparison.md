# Prompt comparison

## Controlled experiment

Both runs used the same 150 tickets, Nebius `openai/gpt-oss-120b`, temperature 0, structured output schema, scoring method, and LangSmith tracing. The only classifier change was the account-workflow disambiguation rule.

| Measure | Original prompt | Improved prompt | Change |
|---|---:|---:|---:|
| Correct | 141/150 | 149/150 | +8 |
| Accuracy | 94.0% | 99.3% | +5.3 percentage points |
| Targeted account-workflow failures | 9 | 0 | -9 |
| Regressions | — | 1 | +1 |

## Fixed cases

The prompt change corrected all nine baseline failures: `t105`–`t107` and `t114`–`t119`. These cover app crashes while viewing account history, rejected newsletter promo codes at checkout, and purchased orders missing from account history.

Representative corrected trace: `t105`, LangSmith run ID `01a07895-78b9-7fb3-ba18-415f5867ff76`. The model explicitly applied the new rule and changed the prediction from `other` to `account_help`.

## Regression

Ticket `t050` changed from the correct `refund_request` prediction to `other`: “Item went on sale right after I ordered. Do you do price adjustments?”

LangSmith run ID: `01a07894-e2e7-7052-9702-6df416728f61`.

The trace reasoning says a post-purchase price adjustment does not fit any defined category. This regression is not directly caused by account-workflow keyword overlap, but it reveals that the original `refund_request` definition does not explicitly cover post-purchase price adjustments. The focused change is still successful against its declared target, but the regression must be reported rather than hidden.
