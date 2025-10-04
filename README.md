## Part 1: Visual Walkthrough

In this part, we explored the AWS Management Console to understand how to set up a Lambda function and concept of events and triggers.

````markdown

In this part, we implemented and tested a Node.js AWS Lambda function.

---

### Step 1: Initial Code Implementation
AWS provides a default Node.js Lambda template with a `TODO implement` comment.  
We modified it as follows:

```javascript
export const handler = async (event) => {
  // TODO implement
  console.log(event);
  
  const response = {
    statusCode: 200,
    body: JSON.stringify('Hello from Lambda!'),
  };
  return response;
};
````

This function simply logs the event and returns `"Hello from Lambda!"`.

---

### Step 2: Creating a Test Event

We created a test event named **`e1`** and executed the Lambda function.
The output displayed `"Hello from Lambda!"` in the response.

---

### Step 3: Customizing Event `e1`

We updated the test event `e1` with custom key-value pairs:

```json
{
  "key1": "value1",
  "key2": "2",
  "key3": "3"
}
```

---

### Step 4: Updating the Lambda Code

We modified the Node.js code to process the event values and calculate a sum:

```javascript
export const handler = async (event) => {
  console.log(event);

  // Adding values from event
  const sum = event.key2 + event.key3; 

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

---

### Step 5: Studying the Output

* The function logged the event details to **CloudWatch Logs**.
* CloudWatch allows us to verify logs, debug issues, and monitor execution in real time. *(Use case: Serverless monitoring and debugging)*
* The response now included both a **sum** field and a **message**.

Example output:

```json
{
  "statusCode": 200,
  "body": "{\"sum\":\"23\",\"message\":\"Hello from Lambda!\"}"
}
```

⚠️ Note: Since `key2` and `key3` were strings (`"2"` and `"3"`), the sum operation resulted in string concatenation (`"23"`).
To perform numeric addition, we would need to convert them:

```javascript
const sum = Number(event.key2) + Number(event.key3);
```

## Part 2: Actual Hands-On
# AWS Lambda with S3 Trigger Hands-On

This exercise demonstrates how to integrate **Amazon S3** with **AWS Lambda** to process files uploaded to an S3 bucket and view the output in **CloudWatch Logs**.

---

## Step 1: Setup
- Open two tabs in the same browser window:  
  - **Tab 1:** AWS Lambda Console  
  - **Tab 2:** Amazon S3 Console  
- Create a text file on your system (e.g., `test.txt`) to upload later.  

---

## Step 2: Create S3 Bucket
1. Go to **S3 Console** → Create a new bucket.  
2. In **Properties**, scroll down to **Event notifications** and check if any event is attached.  

---

## Step 3: Create Lambda Function Using S3 Blueprint
1. In the **Lambda Console**, click **Create Function**.  
2. Select **Use a blueprint** → choose **`s3-get-object` (Python)**.  
3. Execution role: choose the **first option** (new role with basic Lambda permissions).  
4. ⚠️ Do not add the S3 trigger here — we will attach it from the S3 bucket.  

---

## Step 4: Modify Lambda Code
Update the **try block** in the generated code:  

```python
try:
    response = s3.get_object(Bucket=bucket, Key=key)
    print(response)
    print(response['Body'].read())
    # return response['ContentType']
````

* Comment out the first line of `lambda_handler` if it references `event['Records'][0]`.

---

## Step 5: Add S3 Event Notification

1. In **S3 Console**, open your bucket.
2. Go to **Properties → Event notifications**.
3. Create a new event:

   * **Event type:** For all events(first option)
   * **Destination:** Your Lambda function
4. Save changes → S3 is now linked to Lambda.

---

## Step 6: Test Integration

1. Upload a file (e.g., `test.txt` or an image) into your S3 bucket.
2. Open **CloudWatch Logs** → check that your Lambda printed the object details and file body.

---

## Step 7: Fix Permission Issues

If you encounter **Access Denied**:

1. Go to **S3 → Permissions**.
2. Turn off **Block public access (bucket settings)**.
3. Add a **Bucket Policy** (replace bucket name with yours):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Statement1",
      "Principal": "*",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::lambdas3trigger0007/*"
    }
  ]
}
```

4. Save → Re-upload file.
5. Now you should see the file content in **CloudWatch Logs**.

---

**Outcome:**

* Successfully connected **S3 bucket → Lambda function → CloudWatch logs**.
* Lambda can now automatically read and process file contents uploaded to S3.

```

* **S3 bucket event → Lambda trigger → CloudWatch logs output.**
  This demonstrates how AWS Lambda can be used for **event-driven serverless processing**.

```


