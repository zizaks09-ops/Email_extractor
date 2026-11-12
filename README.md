# Matter Intake Extractor

A small working tool that turns an attorney's request to open a new employment matter into a structured intake record, ready for review and for downstream systems.

## The problem

When an attorney emails a paralegal asking to open a new matter, someone reads the email and retypes the key details (client, opposing parties, office, jurisdiction, urgency) into the matter management system, the document management system and the conflicts search. The retyping is slow, and a missed opposing party can cause a missed conflict.

## What it does

1. Takes the text of an intake email.
2. Sends it to a large language model with fixed extraction rules.
3. Returns the details two ways:
   - a **table** for a person to review, with anything uncertain highlighted;
   - **JSON** with fixed keys, so the output can feed SharePoint, iManage and a conflicts search without retyping.
4. Lists **flags for review**: anything missing, uncertain or open to more than one reading goes to a person instead of being guessed.

### JSON output shape

```json
{
  "client": "",
  "opposing_parties": [],
  "matter_type": "",
  "office": "",
  "jurisdiction": "",
  "urgency": "",
  "deadline": "",
  "requesting_attorney": "",
  "flags_for_review": []
}
```

## Extraction rules

1. Use only information stated in the email. Never guess or fill in missing details.
2. If something is missing, write "Not stated."
3. If something is uncertain or has more than one possible answer, write "Unclear" and explain why in the flags.
4. Do not assume the jurisdiction from the office location.
5. Opposing parties feed the conflicts check, so never leave anyone out. Include anyone who is, or may become, an opposing party, mark uncertain ones "(possible)", and flag them.

## How it was tested

Five fictional intake emails (see `sample-emails.md`), each checked against a hand-written answer key. Emails 1 to 3 are straightforward. Emails 4 and 5 are deliberately messy: a vague name, a missing jurisdiction, two possible states and a party who "may" be added.

### What failed, and the fix

On the first run of Email 5, the tool **dropped Dave Russo**, a supervisor the employee's lawyer "may" name personally. The rule "use only information stated" was applied too strictly, so an uncertain party was treated as no party.

For a conflicts check that is the wrong trade-off: a missed party is a real risk, while an extra one costs nothing. Rule 5 was added so possible parties are included, marked "(possible)" and flagged. On re-test, Russo appeared as a possible party with a flag explaining why.

## The human checkpoint

The tool never opens a matter or clears a conflict. Everything it extracts is reviewed by a person, and every flagged item must be resolved before the matter moves forward.

## Where this would go next

- Trigger automatically when an intake email arrives (for example with Power Automate).
- Write the JSON to a SharePoint tracking list and create the iManage workspace from a template.
- Send the opposing parties to the firm's conflicts system, keeping the clearance decision with the conflicts team.
- Grow the test set and re-run it after every prompt or model change.

## Files

| File | What it is |
| --- | --- |
| `index.html` | The tool page. It runs inside Claude as a published artifact, which supplies the model call. |
| `prompt.md` | The full extraction prompt, usable in any chat model. |
| `sample-emails.md` | The five fictional test emails and their answer key. |

All names and matters in this repository are fictional.
