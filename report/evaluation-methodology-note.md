# Evaluation methodology note

The 29 failures in the provided workbook and the nine failures in our reproducible baseline come from two different model runs on the same 150-ticket dataset.

- The instructor-provided workbook records 121 correct predictions out of 150, or 80.7% accuracy. Its 29 failures were used for qualitative error analysis: annotating individual errors, grouping recurring patterns, and selecting `Account_Workflow_Missed` as the improvement target.
- Our reproducible baseline uses the instructor's original classifier prompt with Nebius `openai/gpt-oss-120b`. It records 141 correct predictions out of 150, or 94.0% accuracy, with nine failures. All nine are account-workflow errors.

The earlier analysis is therefore not incorrect. It describes the supplied reference run, while the new baseline measures our controlled experiment. The fact that every failure in the Nebius baseline belongs to the selected account-workflow area further supports the chosen improvement target.

To isolate the effect of the prompt change, the improved result must be compared only with the 94.0% Nebius baseline. Both runs must use the same 150 tickets, Nebius model, model settings, evaluation code, and scoring method; only the classifier prompt should change. The instructor's 80.7% reference result should not be compared directly with the improved Nebius result because that would mix results from different model configurations.
