# Support chatbot with memory

A chat agent for a small shop. It remembers the last 12 messages of the session, uses a calculator for shipping math, and can look up a public fact on Wikipedia without treating that fact as store policy.

## Use case

The hosted n8n chat panel is the front door. A customer can ask about shipping, the free-shipping threshold, returns, and password resets, then ask a follow-up ("what if the order is 60?") and the earlier total is still in memory. Order-specific questions stop at "share the order id and a person will look it up" so the model does not invent a tracking number.

The sample business is Northwind Goods. Replace the facts in the system message before you point real customers at it.

## Required credentials

| Node | Credential |
| --- | --- |
| OpenAI Chat Model | OpenAI API (`openAiApi`) |

Calculator and Wikipedia do not use a credential. Chat history is stored by n8n's window buffer memory, keyed by the chat session id, on this n8n instance.

## Setup

1. Import `workflow.json`.
2. Attach the OpenAI credential on **OpenAI Chat Model**.
3. Open **Support Agent** and replace the Northwind facts in **System Message** with your own. Keep the instruction that refuses to invent order status.
4. Activate the workflow.
5. Open the chat from **Chat**. `public` is off, so the chat is available in the editor rather than on an open URL. Turn `public` on if you want the hosted page, and treat that page as unauthenticated.

## How to customize

- `contextWindowLength` on **Conversation Memory** is 12. Raise it for longer cases. Session id type is **Connected Chat Trigger Node** (`fromInput`), so each chat session keeps its own transcript.
- `maxIterations` is 6. The agent has to have a tool connected. The calculator covers price questions. Remove Wikipedia if you do not want any answer to leave the system message.
- Temperature is 0.2. Lower it if you want the policy wording to stay tighter.
- To hand the conversation to a person, add a tool or a branch that posts the session to Slack when the customer asks for a human. The system message already tells the agent to stop at order lookups.

## Test

In the chat panel, ask "Is shipping free on a 60 dollar order?" The agent should use the calculator and say no, because the threshold is 75. Then ask "What would make it free?" without repeating the number. The answer should still use 60 and 75. Ask for the tracking number of order 1001 and confirm it asks for a person instead of inventing a status. Start a new chat session and confirm it does not remember the previous total.
