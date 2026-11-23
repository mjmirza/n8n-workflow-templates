# Pagination with the HTTP Request node

List endpoints rarely return every row in one response. They return a page, plus a way to ask for the next page: a page number, an offset, or a URL. The HTTP Request node can follow those pages for you. Call the API once without pagination first, and look at the body, so you know which of those shapes you are dealing with.

## Turn pagination on

1. Add an **HTTP Request** node and set the method and URL you already tested.
2. Open **Add option** and choose **Pagination**.
3. Set **Pagination Mode**.

Two modes cover most APIs:

- **Update a Parameter in Each Request** when you must send a changing query or body field (`page`, `offset`, `cursor`).
- **Response Contains Next URL** when the body includes a full URL for the next page.

While you are learning the response, also enable **Response** → **Include Response Headers and Status** so you can see status codes. Remove it later if downstream nodes expect the body alone.

The node exposes `$pageCount` (how many pages have been requested so far, starting at 0), `$request`, and `$response`. `$response.body`, `$response.headers`, and `$response.statusCode` refer to the latest page.

## Page or offset parameters

For an API that takes `?limit=100&offset=0`:

1. Set a fixed query parameter `limit` to `100` in the normal query parameters section.
2. In pagination, choose **Update a Parameter in Each Request**.
3. Add a parameter of type **Query**, name `offset`, value:

```text
{{ $pageCount * 100 }}
```

4. Set **Pagination Complete When** to **Other** and use an expression that is true on the last page. If the body is `{ "data": [ ... ] }` and the last page is a short or empty array:

```text
{{ ($response.body.data ?? []).length < 100 }}
```

5. Set **Limit Pages** to a number you are willing to fetch while testing, such as `3`. Raise it, or clear it, only after the first pages look right.
6. Set **Interval Between Requests** to a few hundred milliseconds if the vendor documents a rate limit. `200` is a modest start.

For a 1-based page number instead of an offset, send `page` as `{{ $pageCount + 1 }}` and stop when the returned array is empty.

## Cursor parameters

For a body shaped like `{ "data": [...], "meta": { "next_cursor": "abc" } }`:

1. Pagination mode: **Update a Parameter in Each Request**.
2. Query parameter name `cursor`, value `{{ $response.body.meta.next_cursor }}`.
3. Complete when `{{ !$response.body.meta.next_cursor }}`.

The first request has no previous response. Leave the parameter unset on that first call if the API rejects an empty cursor. The pagination parameter value can be an expression that sends nothing when `$pageCount` is 0 and the cursor otherwise. If the API insists on omitting the parameter entirely on the first call, make the first request without pagination, then loop with an IF node while a cursor is present. The built-in paginator is the better default when the API accepts a missing or empty cursor.

## Next-URL responses

When the body contains `paging.next` as an absolute URL:

1. Choose **Response Contains Next URL**.
2. Set **Next URL** to `{{ $response.body.paging.next }}`.
3. Complete when that field is empty: `{{ !$response.body.paging.next }}`.

n8n requests that URL as the next page. Do not also increment `offset` yourself.

## Turn pages into one item per record

Pagination returns one item per page, and each item is that page's response. A later node that expects one customer per item will not get that until you split the array.

Add a **Split Out** node after the HTTP Request node and set the field to the array (`data`, `results`, or whatever the sample response uses). The nodes after Split Out then run once per record.

If you need the pages kept together, skip Split Out and operate on `$json.data` in the next node.

## Stop conditions that avoid a runaway loop

Always set a complete-when expression and a page limit during testing. An expression that never becomes true will follow pages until the node times out or the API blocks you. Two expressions that are easy to get wrong:

- Comparing length to `0` when the last page is merely shorter than `limit`. Use `< 100` (the page size), not `=== 0`, unless you have seen a trailing empty page.
- Reading the wrong field, such as `$response.body.next` when the sample shows `meta.next_cursor`. The node then never advances and you fetch the first page until the limit.

After a test run, open the HTTP Request node's output and confirm the item count matches the number of pages you expected. Then open Split Out and confirm the record count matches a known total from the API. Only then raise **Limit Pages** to the real ceiling for that workflow.
