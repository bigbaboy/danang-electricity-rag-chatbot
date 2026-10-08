# Validation Scope

## Evidence currently present

`test_tinh_toan.py` contains pytest tests for calculation and helper behavior. This documentation does not assert that the tests pass in a clean environment.

| Area | Examples represented in the test code |
| --- | --- |
| Electricity billing | Tier allocation, zero use, fractional use, invalid negative input and VAT |
| Household splitting | One-household behavior, comparisons and invalid household counts |
| Consumption | Appliance calculation and zero quantity |
| Solar | Roof-area cases, production assumptions and self-consumption parameters |
| Question handling | kWh extraction, billing-intent detection and answer formatting |
| Helper validation | Month validation, retrieval-score boundaries and HTML escaping |

## Reproduce a result

After installing application dependencies and pytest, run:

```bash
python -m pytest test_tinh_toan.py -v
```

Record the commit, Python version, installed dependency versions and complete test output before reporting pass counts. These tests do not constitute user acceptance testing by EVN.

## Proposed evaluation, not completed work

For the RAG assistant, prepare reference questions linked to source pages and include questions outside the document scope. Evaluate retrieval relevance, answer correctness, citation support and abstention separately. Keep generated answers and human review results so any accuracy claim is reproducible.

For UI validation, check document upload, missing API configuration, empty retrieval and invalid calculation inputs. Mark these checks as completed only after running them.
