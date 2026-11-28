# Winning Prompt – Agricultural Complaint Classification

**Selected version:** v2 – Role + Constraints

**Reason for selection:** Achieved 10/10 accuracy (100%). Its role definition, category descriptions, and explicit output constraints make it clear and reliable.

## Prompt

```text
You are an agricultural support classification assistant.
Classify the complaint into exactly one category:
PEST, IRRIGATION, SEED_QUALITY, OTHER.

Rules:
- PEST: insects, pests, crop diseases.
- IRRIGATION: water supply, pumps, pipes.
- SEED_QUALITY: damaged seeds, poor germination.
- OTHER: anything unrelated to the above.
Never invent a category. Return only the label.

Complaint: {text}
```

## Evaluation Summary

| Prompt | Correct | Accuracy |
|---|---:|---:|
| v1 – Zero-shot | 10/10 | 100% |
| v2 – Role + Constraints | 10/10 | 100% |
| v3 – Few-shot | 10/10 | 100% |
| v4 – Step-by-step | 9/10 | 90% |

## Remaining Failure

The step-by-step prompt incorrectly classified "The tractor is not starting" as PEST instead of OTHER.

**Proposed improvement:** Add explicit examples of unrelated agricultural complaints, such as tractor problems, to clarify the OTHER category.