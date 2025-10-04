# AWS Lambda — S3 Trigger Walkthrough and Hands‑On

This document provides a clear, step‑by‑step walkthrough for exploring AWS Lambda in the Management Console, testing Node.js Lambda functions with custom events, and integrating an S3 bucket with a Lambda function to process uploaded files and view output in CloudWatch Logs.

## Table of Contents

1. [Overview](#overview)
2. [Part 1 — Visual Walkthrough (Node.js Lambda)](#part-1---visual-walkthrough-nodejs-lambda)

   * [Step 1: Initial Code Implementation](#step-1-initial-code-implementation)
   * [Step 2: Creating a Test Event](#step-2-creating-a-test-event)
   * [Step 3: Customizing Event `e1`](#step-3-customizing-event-e1)
   * [Step 4: Updating the Lambda Code (sum example)](#step-4-updating-the-lambda-code-sum-example)
   * [Step 5: Studying the Output](#step-5-studying-the-output)
3. [Part 2 — Hands‑On: AWS Lambda with S3 Trigger](#part-2---hands-on-aws-lambda-with-s3-trigger)

   * [Prerequisites](#prerequisites)
   * [Step 1: Setup](#step-1-setup)
   * [Step 2: Create S3 Bucket](#step-2-create-s3-bucket)
   * [Step 3: Create Lambda Function Using S3 Blueprint](#step-3-create-lambda-function-using-s3-blueprint)
   * [Step 4: Modify Lambda Code](#step-4-modify-lambda-code)
   * [Step 5: Add S3 Event Notification](#step-5-add-s3-event-notification)
   * [Step 6: Test Integration](#step-6-test-integration)
   * [Step 7: Fix Permission Issues](#step-7-fix-permission-issues)
4. [Outcome / Expected Behavior](#outcome--expected-behavior)
5. [Troubleshooting](#troubleshooting)

---

## Overview

This README documents two parts: (1) a visual walkthrough and simple Node.js Lambda examples demonstrating how events are received and logged, and (2) a practical hands‑on exercise that links an S3 bucket to a Lambda function to automatically process uploaded objects and emit output to CloudWatch Logs.

---

## Part 1 — Visual Walkthrough (Node.js Lambda)

### Step 1: Initial Code Implementation

AWS Lambda provides a default Node.js template containing a `TODO implement` comment. Replace it with a minimal handler that logs the incoming event and returns a basic response:

```javascript
export const handler = async (event) => {
  // Log the incoming event for debugging and verification
  console.log(event);

  const response = {
    statusCode: 200,
    body: JSON.stringify('Hello from Lambda!'),
  };

  return response;
};
```

### Step 2: Creating a Test Event

1. In the Lambda console, open the function and select **Test**.
2. Create a new test event with a name such as `e1` and leave the default template or provide custom JSON.
3. Execute the function using the test event — the response should contain `"Hello from Lambda!"`.

### Step 3: Customizing Event `e1`

Example custom event JSON used in the walkthrough:

```json
{
  "key1": "value1",
  "key2": "2",
  "key3": "3"
}
```

### Step 4: Updating the Lambda Code (sum example)

To demonstrate event processing, update the handler to compute a `sum` from values in the event and return it in the response:

```javascript
export const handler = async (event) => {
  console.log(event);

  // If key2 and key3 are strings, the + operator will perform string concatenation.
  // Convert to Number for numeric addition if required.
  const sum = Number(event.key2) + Number(event.key3);

  const response = {
    statusCode: 200,
    body: JSON.stringify({
      sum: sum,
      message: 'Hello from Lambda!'
    }),
  };

  return response;
};
```

**Note:** If you omit `Number(...)` and `key2`/`key3` are strings, `event.key2 + event.key3` will concatenate resulting in `"23"` for inputs `"2"` and `"3"`.

### Step 5: Studying the Output

* Lambda function execution logs are available in **CloudWatch Logs**. Use the logs to verify event contents, debug errors, and confirm the function's behavior.
* Example function response (stringified `body` field):

```json
{
  "statusCode": 200,
  "body": "{\"sum\":5,\"message\":\"Hello from Lambda!\"}"
}
```

*(The `body` field is a JSON string; when consumed by other services, parse it accordingly.)*

---

## Part 2 — Hands‑On: AWS Lambda with S3 Trigger

### Prerequisites

* An AWS account with permissions to create S3 buckets, Lambda functions, and IAM roles.
* Access to the AWS Management Console.
* Basic familiarity with the Lambda console, S3 console, and CloudWatch Logs.

### Step 1: Setup

Open two browser tabs in the same window for convenience:

* Tab 1: AWS Lambda Console
* Tab 2: Amazon S3 Console

Prepare a small file to upload (for example, `test.txt`) to use during testing.

### Step 2: Create an S3 Bucket

1. In the S3 console, create a new bucket using a globally unique name.
2. Review the bucket **Properties** and note the **Event notifications** section (it will be updated later when attaching the Lambda function).

### Step 3: Create Lambda Function Using S3 Blueprint

1. In the Lambda console click **Create function**.
2. Select **Use a blueprint**, then choose `s3-get-object` (Python) as the blueprint.
3. For the execution role, select the option to create a new role with basic Lambda permissions (or use an existing role with `AWSLambdaBasicExecutionRole` and necessary S3 read permissions).

**Important:** Do not attach the S3 trigger from the Lambda console at this step — the walkthrough attaches the trigger from the S3 bucket side.

### Step 4: Modify Lambda Code

In the generated Python code, update the `try` block where the function reads the S3 object so it prints the object metadata and body. Example modification:

```python
try:
    response = s3.get_object(Bucket=bucket, Key=key)
    print(response)
    print(response['Body'].read())
    # return response['ContentType']
```

If the generated handler references `event['Records'][0]` and you need a different approach for testing, comment or adjust that line accordingly.

### Step 5: Add S3 Event Notification

1. In the S3 console, open the bucket you created.
2. Go to **Properties → Event notifications**.
3. Create a new event notification with these settings:

   * **Event type:** All object events (or choose the specific event types you need, e.g., `PUT`).
   * **Destination:** Lambda function (select the Lambda created earlier).
4. Save the notification. S3 and Lambda are now linked: object uploads will trigger the Lambda function.

### Step 6: Test Integration

1. Upload a test file to the bucket (for example, `test.txt` or an image).
2. Open **CloudWatch Logs** and locate the log stream for the Lambda function.
3. Verify that the function printed the object metadata and the file contents as expected.

### Step 7: Fix Permission Issues

If the Lambda function receives an `AccessDenied` error when attempting to fetch the object from S3, take the following corrective steps:

1. In the S3 console, under **Permissions**, verify the bucket's public access settings. For many workflows you should keep block public access enabled and instead grant precise permissions using IAM roles or bucket policies.
2. If you intend to grant public GET access for testing (not recommended for production), disable block public access settings and add a bucket policy such as the example below (replace the bucket name accordingly):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Statement1",
      "Principal": "*",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

3. Preferred approach: update the Lambda execution role to include permissions allowing `s3:GetObject` on the target bucket. For example, attach a policy granting `s3:GetObject` for `arn:aws:s3:::your-bucket-name/*` to the Lambda role.
4. Re‑upload the test file and confirm the Lambda now reads the object successfully.

---

## Outcome / Expected Behavior

* Uploading an object to the configured S3 bucket triggers the Lambda function.
* The Lambda function retrieves the object using the S3 API, prints metadata and the object body to CloudWatch Logs, and can process the file programmatically.
* This pattern demonstrates event‑driven serverless processing (S3 event → Lambda handler → CloudWatch Logs).

---

## Troubleshooting

**Symptom: No logs in CloudWatch after uploading a file**

* Confirm the event notification is configured in the S3 bucket and points to the correct Lambda function.
* Verify the Lambda execution role is valid and has `AWSLambdaBasicExecutionRole` permissions to write logs to CloudWatch.
* Check CloudWatch Log Group retention and permissions.

**Symptom: `AccessDenied` when Lambda calls `s3.get_object`**

* Ensure the Lambda execution role has an IAM policy that grants `s3:GetObject` on the bucket object ARNs.
* Avoid opening the bucket publicly unless absolutely necessary; prefer granting least privilege to the Lambda role.

---



