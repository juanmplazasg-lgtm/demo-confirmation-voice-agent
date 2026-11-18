# Demo confirmation voice agent (prototype)

A voice agent that calls a lead to confirm a booked product demo. It says it is an AI, handles
three outcomes (confirm, reschedule, cancel) and saves a structured result for each call, so the
outcome can be written back to a CRM.

**Status: prototype in development.** Tested on web calls with made-up lead data. It has not
called real leads and it is not connected to a phone number.

## How it works

1. A page (or a CRM job) starts a call and passes the lead's details as variables:
   `name`, `company`, `demo_date`, `demo_time`.
2. The agent opens with those details and asks if the time still works.
3. It follows one of three paths: confirm, reschedule (it repeats the new time back and promises
   nothing about availability), or cancel (it asks once for a reason).
4. When the call ends, Vapi stores a structured result that follows a JSON schema:

```json
{ "outcome": "rescheduled", "requested_new_time": "Friday at 10 in the morning",
  "cancel_reason": "", "do_not_call": false }
```

## Guardrails in the prompt

- It says early that it is an AI assistant, and says so again if asked.
- It does not sell, quote prices or describe features.
- It does not ask for personal information.
- If the person asks not to be called again, it records `do_not_call: true` and ends the call.
- After two failed attempts to understand, it hands off to a person by email.

## Files

| File | What it is |
|---|---|
| `assistant.json` | The full assistant definition: prompt, first message, model, voice, transcriber and the structured output schema. |
| `test-page/index.html` | A single-file test page. It starts a web call with the Vapi Web SDK, passes the lead variables and shows the live transcript. |

## Run it

1. Create the assistant: `POST https://api.vapi.ai/assistant` with `assistant.json` as the body
   and your private key as a bearer token.
2. Put the returned id in `test-page/index.html` (`YOUR_ASSISTANT_ID`).
3. Serve the page locally: `python3 -m http.server 8765 --bind 127.0.0.1` inside `test-page/`.
4. Open `http://localhost:8765`, paste your Vapi public key and start the call. The key stays in
   the tab and is never saved.

## Stack

Vapi (orchestration) · OpenAI GPT-4.1 (dialogue) · Deepgram Nova-3 (speech to text) · Vapi voice
(text to speech) · plain HTML and JavaScript for the test page.

## Known limits and next steps

- Turn-taking uses a fixed silence delay, so the agent sometimes talks over the caller.
- On the confirm path the closing sentence can be cut when the call ends. Next change: end the
  call on a spoken phrase.
- Next: write the outcome back to the CRM through a webhook, then outbound calls on a real number.
