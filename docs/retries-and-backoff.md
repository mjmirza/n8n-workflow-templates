# Retries and backoff

Outside services fail in ways that a second attempt will fix: a connection reset, a 503, a 429 that asks you to slow down. They also fail in ways a retry will not fix: a 400 because the body is wrong, or a 401 because the token is revoked. Retry only the first kind.

## Use the node's built-in retry for short blips

Each node has **Settings** with **Retry On Fail**.

1. Open the node settings.
2. Turn **Retry On Fail** on.
3. Set **Max Tries** to the total number of attempts. The default is 3, which is enough for a brief network error.
4. Set **Wait Between Tries (ms)** to a fixed pause. `1000` waits one second between attempts. `5000` is a reasonable start for an API that rate-limits lightly.

The wait is a fixed delay, not a backoff. The node retries the same input. If the node has side effects that are not idempotent, a retry can create a duplicate charge, ticket, or message. Prefer this setting for read requests and for writes that the API can deduplicate with an idempotency key.

Leave **Retry On Fail** off on nodes whose errors are bugs in your mapping. Retrying a "column does not exist" error three times only delays the failure.

## Add exponential backoff when the API tells you to wait

When a vendor documents rate limits, or you see 429 responses under load, use a loop and a **Wait** node so the delay grows.

1. Put a **Set** node in front of the call that initializes `attempt` to `0`.
2. On the HTTP Request node, include the response status, and set **On Error** to **Continue (using error output)** so a thrown error is visible as data. If you use **Never Error** instead, read `statusCode` from the main output.
3. Add an **IF** node. The success branch is a status from 200 to 299. The failure branch continues.
4. On the failure branch, add a **Code** node that computes the next delay and gives up after a cap:

```javascript
const attempt = Number($json.attempt ?? 0);
const status = Number($json.statusCode ?? 0);
const retryable = status === 429 || status >= 500 || status === 0;
if (!retryable || attempt >= 5) {
  return { json: { ...$json, retry: false } };
}
const delaySeconds = Math.min(60, 2 ** attempt);
return {
  json: {
    ...$json,
    retry: true,
    attempt: attempt + 1,
    delaySeconds,
  },
};
```

5. An **IF** on `retry` equal to `true` leads to a **Wait** node set to `{{ $json.delaySeconds }}` seconds, then back to the HTTP Request node. The false branch leads to a **Stop and Error** node whose message includes the status code.

The delays for attempts 0 through 4 are 1, 2, 4, 8, and 16 seconds, then the `Math.min` cap holds them at 60. Five retries is a ceiling, not a target. Stop earlier if the API returns a `Retry-After` header you can parse into the wait.

## Respect Retry-After

If the error output keeps response headers, read the header before you invent a delay:

```javascript
const header = $json.headers?.['retry-after'];
const seconds = Number(header);
const delaySeconds = Number.isFinite(seconds) && seconds > 0
  ? Math.min(120, seconds)
  : Math.min(60, 2 ** Number($json.attempt ?? 0));
```

Header names are often lowercased by the HTTP Request node. Check a real 429 response once and use the name you see.

## Do not stack two retry policies

If the node has **Retry On Fail** enabled and the workflow also loops, one outage becomes many more requests than you think. Use the built-in retry for transport errors on idempotent reads. Use the explicit loop when you need backoff, a cap, or a branch that alerts. Not both on the same call.

## See the attempts in the execution

A fixed node retry does not add a new execution. It shows as one node that took longer. A Wait loop shows each attempt as nodes you can open. While you are proving the loop, temporarily set the delays to one second and force a failure with a URL that returns 500, then restore the real URL and the real delays before you activate the workflow.

Queue-mode workers have their own job timeouts. A Wait of many minutes is fine, but a retry loop that can run for an hour will hold a worker slot. Keep the cap small, and send a final failure to the error workflow instead of retrying without a limit.
