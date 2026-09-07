# Baseline LangSmith trace review

## Passing example: t000

- True category: `order_status`
- Predicted category: `order_status`
- Result: correct
- LangSmith trace ID: `01a07861-725f-70c3-bbb1-e24cae491935`
- Ticket: "Hi, I ordered a blender 5 days ago and the tracking page hasn't updated. Can you tell me where it is?"
- Model reasoning: "The customer is asking about the location and tracking of a recent order."
- Interpretation: The original prompt explicitly associates tracking and order location with `order_status`, so the model follows the intended route.

## Failed example: t105

- True category: `account_help`
- Predicted category: `other`
- Result: incorrect
- LangSmith trace ID: `01a07862-7f92-7622-93a9-f042272b577e`
- Ticket: "Your app keeps crashing whenever I try to view my order history."
- Model reasoning: "The issue is about the app crashing when accessing order history, which is a technical problem not covered by the defined categories."
- Interpretation: The model correctly identifies an app-access problem but routes it to `other` because the original `account_help` definition only lists login, password, address, and payment-method changes. The prompt does not say that failures accessing account-linked order history belong to `account_help`.

## Trace structure

Both examples completed successfully at the system level and contain five linked spans: the ticket-level evaluation run, prompt template, runnable sequence, model call, and structured-output parsing step. The failed classification is therefore a prompt-policy failure, not an API, parsing, or execution failure.

This evidence directly supports the proposed account-workflow disambiguation rule. After the improved run, these baseline trace IDs should be compared with the corresponding improved trace IDs for the same tickets.

## Paired trace comparison after the prompt change

### Rejected promo code: t114

- Baseline trace: `01a07862-95f5-71b0-942c-56c3228f18d8`
- Baseline prediction: `other` (incorrect)
- Baseline reasoning: The discount-code problem did not fit any category listed in the original prompt.
- Improved trace: `01a07895-90cf-7651-99c3-1c47ddcfa216`
- Improved prediction: `account_help` (correct)
- Improved reasoning: The model explicitly applied the new rejected-promo-code-at-checkout rule.

### Missing account-history record: t117

- Baseline trace: `01a07862-9e57-7821-ab3f-e03222f04215`
- Baseline prediction: `order_status` (incorrect)
- Baseline reasoning: The model treated the missing account record as a request to locate or track an order.
- Improved trace: `01a07895-98e2-7f62-94b4-147c5a6e062b`
- Improved prediction: `account_help` (correct)
- Improved reasoning: The model explicitly applied the new missing-order-record-in-account-history rule.

Both paired examples contain five successful spans in each run. The execution path and output parsing remained healthy; the changed classifications came from the revised prompt policy.
