# Redact personal data before an LLM call

Strips common personal-data patterns out of a document, runs the requested task on the redacted text, and drops the model output if a pattern still shows up.

## Use case

Support notes, HR excerpts, and pasted emails should not be sent to a model in the clear. This workflow replaces emails, US phone numbers, Social Security numbers, card numbers that pass the Luhn check, and IPv4 addresses with numbered tokens such as `[EMAIL_1]`. The token map is counted and then discarded. It is not sent to the model and it is not included in the HTTP response, so the call cannot be undone by stuffing the originals back in.

## Required credentials

| Node | Credential |
| --- | --- |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |

## Setup

1. Import `workflow.json`.
2. Attach the OpenAI credential.
3. Activate the workflow. The webhook path is `pii-redaction`.

## Input

```json
{
  "task": "Summarize the customer's issue in three bullets.",
  "text": "Amina Khan (amina@example.com, 415-555-0134) was double charged. SSN on the form was 123-45-6789."
}
```

`body` and `document` are accepted aliases of `text`. If `task` is omitted, the workflow summarizes the document.

The response always includes `blocked`, `redactionCounts`, and either `result` (the model text) or `result: null` plus `leaks` and a `reason`. When `blocked` is true, the model text is not returned.

## How to customize

- Add patterns in **Redact PII** and the same patterns in **Scan Model Output**. Keep the two lists aligned. The scanner is what stops a model from echoing a value the first pass missed.
- Card numbers are only removed when they pass Luhn, so order ids made of digits stay in the document.
- Change the default task string in **Redact PII**.
- 20,000 characters is the input cap. Raise the `slice` if your documents are longer and your model context allows it.
- This is a pattern filter, not a full recognizer for names or addresses. Put a second model pass in front of **Run Task On Redacted Text** if you need names removed too, and still keep the deterministic scan after the task.

## Test

Send the sample above. The text that reaches the model should contain tokens instead of the email, phone, and SSN, and `redactionCounts` should be non-zero. Confirm the HTTP body does not contain `amina@example.com`. Then temporarily point the task at "repeat the document verbatim" and confirm a leak is blocked rather than returned.
