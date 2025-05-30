---

copyright:
   years: 2025
lastupdated: "2025-05-13"

keywords: tutorial, multipart upload, cloud object storage

subcollection: cloud-object-storage

content-type: tutorial
services:  # Only if the tutorial includes multiple services. If it only uses your service, don't specify. DO NOT set any platform subcollections.
account-plan: paid # Set `lite` if tutorial can be completed by using only Lite plan services; Set `paid` if the tutorial requires a pay-go or subscription versions of plans for the service
completion-time: 1h # Estimated time to complete the steps in this tutorial. Minute values are supported up to 90 minutes. Whole hours are also supported; for example: 2h

---

{{site.data.keyword.attribute-definition-list}}

# Using cURL to perform a multipart upload
{: #tutorial-curl-multipart}
{: toc-content-type="tutorial"}
{: toc-completion-time="1h"}

In this tutorial, you learn how to upload large files in multiple parts by using cURL commands on a Linux operating system.  Steps include running split in a terminal window for separating the file into parts, initiating the upload, uploading the parts, and completing the upload.  In addition, this tutorial uses a script to upload each file part and get its entity tag (ETag) and to generate the XML block that is needed to complete the multipart upload operation.
{: shortdesc}

## Scenario
{: #curl-multipart-scenario}

Most tools, such as the CLIs or the {{site.data.keyword.cloud}} console, as well as most compatible libraries and SDKs, automatically transfer objects in multipart uploads.  Refer to [Storing large objects](/docs/cloud-object-storage?topic=cloud-object-storage-large-objects) for details about multipart upload operations.  For some scenarios, you might prefer to use cURL commands for a multipart upload, for example, if you are working within a read-only operating system in which you cannot install the CLI.

## Before you begin
{: #curl-multipart-prereqs}

Before you begin the upload, you need:
* An [{{site.data.keyword.cloud_notm}} account](https://cloud.ibm.com)
* An [instance of {{site.data.keyword.cos_full_notm}}](/docs/cloud-object-storage?topic=cloud-object-storage-provision)
* A bucket
* Writable free space on your local system that is greater than the total file size of the upload

Using `curl` assumes a certain amount of familiarity with the command line and {{site.data.keyword.cloud}}, and have the necessary information from a [service credential](/docs/cloud-object-storage?topic=cloud-object-storage-service-credentials), the [endpoints reference](/docs/cloud-object-storage?topic=cloud-object-storage-endpoints), or the [console](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage).

## Split the file into parts
{: #curl-multipart-split}
{: step}

For multipart uploads, every file part (except for the last part) must be at least 5 MB.  To split a file into separate parts, you can run split in a terminal window.  When you decide how to split the file given its total size, keep in mind that there is a maximum of 10,000 (10k) parts for a multipart upload.

In the following example, the split command uses the -b option with value of 100M, a file that is named TESTFILE.iso, and a prefix of "part-".

```split -b 100M TESTFILE.iso part-```
{: pre}

This command generates file parts of the size that is specified, with the names part-aa, part-ab, part-ac, and so on.

## Initiate upload and get UploadID
{: #curl-multipart-initiate}
{: step}

To initiate a new multipart upload, run cURL to issue a `POST` request to an object with the query parameter `uploads`, which creates a new `UploadId` value. You reference the `UploadID` with each part of the object that is being uploaded.

**Example request**
```sh
curl -X "POST" "https://s3.private.us-south.cloud-object-storage.appdomain.cloud/mputest/TESTFILE.iso?uploads"
-H "Authorization: bearer $bearertoken"
```
{: pre}

**Example response**
```xml
_<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<InitiateMultipartUploadResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Bucket>mputest</Bucket>
  <Key>TESTFILE.iso</Key>
  <UploadId>01000194-8fab-0f08-3c54-d7df03b4a127</UploadId>
</InitiateMultipartUploadResult>_
```
{: screen}

From the response, make note of the `UploadID`.  In this example, the `UploadID` value is `01000194-8fab-0f08-3c54-d7df03b4a127`.

## Use a script to upload parts and build XML block
{: #curl-multipart-script}
{: step}

Next, you run cURL to issue a `PUT` request for each file part you want to upload.  The response header for each part upload includes an ETag value, which you must save as input for the final step in the multipart upload operation.

Instead of uploading each part manually, you can use a bash script like the one that follows. The script has two functions:
1.	upload the parts
2.	create the XML part manifest, which is stored locally in the /tmp directory

As each file part is uploaded, the script extracts its ETag from the response header and saves it and the part number into an XML block as input for the final step.

**Example script**



```sh
#!/usr/bin/env bash
set -euo pipefail

# ─── Configuration ───────────────────────────────────────────────────────────────
# Public endpoint for your bucket
ENDPOINT="$COS endpoint"
BUCKET="$bucket"
KEY="$Filename"
UPLOAD_ID="$uploadID gained from initiation of upload"
ACCESS_TOKEN="$BEARERTOKEN"
PARTS_PREFIX="part-"
MANIFEST="parts_manifest.xml"
# ─────────────────────────────────────────────────────────────────────────────────

# 1) Start a fresh manifest
cat > "$MANIFEST" <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<CompleteMultipartUpload>
EOF

part=1
for f in ${PARTS_PREFIX}*; do
  if [[ ! -r "$f" ]]; then
    echo "ERROR: no files matching ${PARTS_PREFIX}*" >&2
    exit 1
  fi

  echo "Uploading $f as part $part…"

  # 2) PUT the part, capture headers
  hdrfile=$(mktemp)
  http_code=$(
    curl -sS \
      -D "$hdrfile" \
      -o /dev/null \
      -X PUT "${ENDPOINT}/${BUCKET}/${KEY}?partNumber=${part}&uploadId=${UPLOAD_ID}" \
      -H "Authorization: Bearer ${ACCESS_TOKEN}" \
      --data-binary @"${f}" \
      -w '%{http_code}'
  )

  if [[ "$http_code" != "200" ]]; then
    echo "ERROR: HTTP $http_code for part $part" >&2
    sed 's/^/  /' "$hdrfile" >&2
    rm -f "$hdrfile"
    exit 1
  fi

  # 3) Extract clean ETag (strip quotes/carriage returns)
  etag=$(grep -i '^ETag:' "$hdrfile" \
         | cut -d' ' -f2 \
         | tr -d '"\r')
  rm -f "$hdrfile"

  if [[ -z "$etag" ]]; then
    echo "ERROR: no ETag for part $part" >&2
    exit 1
  fi

  echo " → Part $part ETag: $etag"

  # 4) Append a well-formed <Part> block
  printf '  <Part>\n    <PartNumber>%d</PartNumber>\n    <ETag>"%s"</ETag>\n  </Part>\n' \
    "$part" "$etag" >> "$MANIFEST"

  part=$((part + 1))
done

# 5) Close the manifest
echo '</CompleteMultipartUpload>' >> "$MANIFEST"

echo "✔ Manifest generated: $MANIFEST"


## Complete multipart upload
{: #curl-multipart-complete}
{: step}

When all parts are finished uploading, you complete the upload by sending a request with the `UploadId` and an XML block that lists each part number and its respective Etag value.
```
{: codeblock}

**Example request**
```sh
curl -X POST "https://s3.private.us-south.cloud-object-storage.appdomain.cloud/mputest/TESTFILE.iso?uploadId=01000194-8fab-0f08-3c54-d7df03b4a127"
 -H "Authorization: Bearer $bearertoken" -H "Content-Type: application/xml" --data-binary @parts_manifest.xml
```
{: pre}

**Example response**
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<CompleteMultipartUploadResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Location>http://s3.private.us-south.cloud-object-storage.appdomain.cloud/mputest/TESTFILE.iso</Location>
  <Bucket>mputest</Bucket>
  <Key>TESTFILE.iso</Key>
  <ETag>&quot;1a90419141160aa6713b8d196773345e-94&quot;</ETag>
</CompleteMultipartUploadResult>
```
{: screen}

## Next steps
{: #curl-multipart-step-next}

If all goes well, you receive a confirmation that the file was uploaded successfully to the wanted bucket. If you encounter errors, be sure to clean up incomplete multipart uploads. If an incomplete multipart upload is not stopped, the partial upload continues to use resources. See [Using cURL](/docs/cloud-object-storage?topic=cloud-object-storage-curl#curl-multipart-get) for the syntax to get or stop incomplete multipart uploads.
