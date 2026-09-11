# Hiver AI Support Agent — AmazonHelp

An evaluation-first AI customer-support agent built for the **Hiver SDE Intern Take-Home Assignment** using real customer-support conversations from Twitter/X.

The system is designed to demonstrate three core capabilities:

1. **Intent Classification** — classifies incoming customer messages into a compact support-intent taxonomy.
2. **Grounded Reply Drafting** — retrieves historically similar AmazonHelp interactions and uses their observed support resolutions as evidence for drafting a response.
3. **Escalation Decision** — determines whether a request can be auto-handled or should be escalated to a human, with an explicit reason.

The project emphasizes **evaluation, leakage prevention, grounding, safety, and failure analysis** rather than optimizing a single headline metric.

---

## Selected Brand

**AmazonHelp (@AmazonHelp)**

The Customer Support on Twitter dataset was analyzed to identify a brand with sufficient interaction volume and diverse customer-support issues.

After preprocessing and constructing customer-to-support interaction pairs, the selected AmazonHelp subset contained:

**168,814 historical customer-support interactions**

AmazonHelp was selected because it provided:

- Large interaction volume
- Broad support-issue coverage
- Repeated resolution patterns
- Sufficient historical responses for retrieval-based grounding

---

# 1. Problem Framing

The objective is to build a measurable prototype for AI-assisted customer support.

For each incoming customer message:

```text
Customer Message
       |
       v
Intent Classification
       |
       v
Historical Resolution Retrieval
       |
       v
Grounded Reply Draft
       |
       v
Escalation Policy
       |
       +---------------------+
       |                     |
       v                     v
  Auto-handle         Human Escalation
