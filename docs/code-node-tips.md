# Code node tips

The Code node is for shaping data: renaming fields, filtering rows, and computing a value from several inputs. It is a poor place to reimplement an HTTP client, a retry loop, or a credential. Those already exist as nodes, and they stay visible on the canvas.

n8n 2.x runs the Code node in a task runner. You cannot import arbitrary npm packages from a workflow. Use JavaScript that ships with the runtime (`JSON`, `Date`, `Intl`, `Array` methods). On n8n 1.x the Code node still runs inside the n8n process and exposes Luxon's `DateTime` global. Do not write a template that depends on `DateTime` unless the README says it targets 1.x.

## Run once for all items, or once per item

The mode is at the top of the node.

**Run Once for All Items** receives the whole input. Read it with `$input.all()` and return an array:

```javascript
return $input.all().map((item) => {
  const email = typeof item.json.email === 'string' ? item.json.email.trim().toLowerCase() : '';
  return {
    json: {
      ...item.json,
      email,
      valid: email.includes('@'),
    },
    pairedItem: item.pairedItem,
  };
});
```

**Run Once for Each Item** runs the code separately for every input item. Read `$input.item.json` and return one object, not an array:

```javascript
const email = String($input.item.json.email ?? '').trim().toLowerCase();
return {
  json: {
    ...$input.item.json,
    email,
    valid: email.includes('@'),
  },
};
```

Returning an array from **Each Item** mode adds a nested list and breaks the next node. Returning a bare object from **All Items** mode throws. Match the return shape to the mode.

`pairedItem` keeps the editor's line from an output row back to the input row that produced it. Copy it through when you map one output per input. When you expand one input into many outputs, set it yourself:

```javascript
return $input.all().flatMap((item, index) => {
  const rows = Array.isArray(item.json.rows) ? item.json.rows : [];
  return rows.map((row) => ({
    json: row,
    pairedItem: { item: index },
  }));
});
```

## Read another node's output by name

`$json` is the current item. To read a node elsewhere on the canvas, call it by the name shown under that node:

```javascript
const baseUrl = $('Config').first().json.baseUrl;
const people = $('Fetch people').all().map((item) => item.json);
```

Renaming the node breaks this reference. Name the source node before you write the code, and avoid names that differ only by a trailing space.

## Guard the fields you expect

APIs omit keys. A Code node that starts with `item.json.user.email.toLowerCase()` fails the whole execution on the first incomplete row. Check the type, and emit a field the next node can branch on:

```javascript
const user = $input.item.json.user;
const email = user && typeof user.email === 'string' ? user.email : null;
return { json: { email, skip: email === null } };
```

Follow the Code node with an **IF** on `skip` if those rows should not continue.

## Dates

Return ISO strings rather than Date objects. A Date does not survive the JSON boundary the way you expect:

```javascript
const created = new Date($input.item.json.createdAt);
const iso = Number.isNaN(created.getTime()) ? null : created.toISOString();
return { json: { createdAt: iso } };
```

In expressions, `$now` is a date/time value. Inside the Code node, use `new Date()`.

## What not to put in the node

- Secrets and tokens. Read those from a credential on an HTTP Request node. `process.env` is blocked by default and should stay blocked, because the same environment holds the instance encryption key.
- Long HTTP calls. You lose pagination, retries, and the credential UI.
- `console.log` of a full request. Execution logs are visible to anyone who can open the workflow.

Keep a Code node short enough to read in the editor without scrolling much. If it grows past a screen, split the steps across two nodes so a failure points at the step that broke.
