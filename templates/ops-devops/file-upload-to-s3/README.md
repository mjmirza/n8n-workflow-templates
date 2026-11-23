# File upload to S3 with metadata

Accepts a multipart upload and stores the file in Amazon S3 under `uploads/<date>/<source>/<timestamp>-<filename>`. The object is private, storage class Standard, and tagged with the source, original filename, content type, and note. The webhook responds with the bucket and key.

The form field that holds the file must be named `file`. Optional text fields:

- `source` — short label used in the key path. Characters other than letters, numbers, `.`, `_`, and `-` are removed.
- `note` — stored as an S3 object tag.

## Credentials

- AWS, on the AWS S3 node: access key, secret, and region with `s3:PutObject` and `s3:PutObjectTagging` on the bucket.

The webhook itself has no authentication. Put it behind a gateway, or set the Webhook node's authentication before you expose it.

## Setup

1. Import `workflow.json`.
2. In **Build object key**, change `incoming-uploads` to your bucket. The S3 node reads the bucket and key from that node.
3. Attach the AWS credential on **Upload to S3**.
4. Activate the workflow and copy the production webhook URL. The path is `uploads`.

## Customize

- **Build object key** is the only place that decides the object key and the tag values.
- Change ACL or storage class on **Upload to S3**.
- Add webhook authentication if callers are not already authenticated by something in front of n8n.

## Test

```bash
curl -F "file=@./report.pdf" -F "source=billing" -F "note=october"   https://<your-n8n-host>/webhook/uploads
```

A successful call returns `{"ok": true, "bucket": "...", "key": "uploads/..." }`. A call without the `file` field returns HTTP 400. In S3, the object tags should show `source`, `original-filename`, `content-type`, and `note`.
