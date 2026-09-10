# Hiver AI Support Agent — AmazonHelp

An AI customer-support agent built for the Hiver SDE Intern Take-Home Assignment using real customer-support conversations from Twitter.

The system performs three core tasks:

1. **Intent Classification** — classifies incoming customer messages into support intents.
2. **Historical Resolution Retrieval** — retrieves similar historical customer-support interactions to ground responses.
3. **Escalation Decision** — determines whether an interaction should be auto-handled or escalated to a human, with an explicit reason.

## Selected Brand

**AmazonHelp (@AmazonHelp)**

The Customer Support on Twitter dataset was analyzed to identify a suitable brand with sufficient customer-support interactions. Customer-to-support conversation pairs were constructed for downstream modeling and retrieval.

## Intent Taxonomy

The system uses 11 support intents:

- `I01_order_delivery`
- `I02_missing_package`
- `I03_return`
- `I04_refund`
- `I05_wrong_damaged_item`
- `I06_payment_billing`
- `I07_account`
- `I08_prime_subscription`
- `I09_cancellation_modification`
- `I10_general_information`
- `I11_other_unclear`

The `Other/Unclear` category is intentionally retained to avoid forcing ambiguous customer messages into an incorrect specific intent.

## System Pipeline

```text
Customer Message
       |
       v
Intent Classification
       |
       +--------------------+
       |                    |
       v                    v
Historical Retrieval    Escalation Policy
       |                    |
       v                    v
Relevant Resolution     Auto-handle /
Evidence                Human Escalation
       |
       v
Draft Support Reply
