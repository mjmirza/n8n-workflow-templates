# Google Sheets row to Jira issue

When a row is added to a Google Sheet, creates a Jira Cloud issue and writes the new issue key back onto that row.

The sheet needs a header row with these columns:

| Summary | Description | Priority | Issue Key | Status |
| --- | --- | --- | --- | --- |
| required | optional | optional, copied into the Jira description | filled by this workflow | set to `Created` |

Rows with an empty Summary are ignored.

## Credentials

- Google Sheets Trigger OAuth2 (the trigger's recommended OAuth2 credential), with access to the spreadsheet.
- Google Sheets OAuth2 on the update node, for the same Google account. The trigger and the update node use different credential types.
- Jira Software Cloud: site, email, and API token.

## Setup

1. Create the sheet tab and the header row above. The first sheet's gid is usually `0`.
2. Import `workflow.json`.
3. On **Google Sheets Trigger**, set **Document** to the spreadsheet id (the long id in the sheet URL) and **Sheet** to the numeric gid.
4. On **Create Jira issue**, set **Project** and **Issue Type** to ids from your Jira site. The template uses project `10000` and issue type `10001`, which are only starting points.
5. On **Write issue key**, set the same spreadsheet id and set **Sheet** to the tab name. The template uses `Inbox`.
6. Attach all three credentials. Execute **Google Sheets Trigger** once so n8n stores the current rows, then activate. Only rows added after activation are processed.

## Customize

- **Create Jira issue** puts Priority and Description into the Jira description field. Map Jira priority or labels in **Additional Fields** if your project requires them.
- **Write issue key** matches `row_number`, which the trigger adds and which is not a real column. Do not create a `row_number` column in the sheet.
- Change **Status** from `Created` if your process uses a different value.

## Test

Append a row with a Summary. After the trigger polls, Jira should contain the issue and the same sheet row should show the issue key and Status `Created`. Append a row with a blank Summary and confirm no issue is created.
