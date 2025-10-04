## Part 1: Visual Walkthrough

In this part, we explored the AWS Management Console to understand how to set up a Lambda function and concept of events and triggers.

````markdown
## Part 2: Actual Hands-On

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


````markdown
##  Part 3: S3 Trigger Hands-On

In this part, we connected **Amazon S3** with **AWS Lambda** to automatically process files uploaded to an S3 bucket.  
The goal was to upload a text file to S3 and read its contents using a Lambda function.

---

### Step 1: Setup
1. Open two tabs in the same browser window:  
   - **Tab 1:** AWS Lambda Console  
   - **Tab 2:** Amazon S3 Console  

2. Create a **text file** of your choice on your local system (e.g., `test.txt`).

---

### Step 2: Create an S3 Bucket
1. Go to the **S3 Console** → Create a new bucket.  
2. In the **Properties** section, scroll to **Event notifications** to check if any event is attached.  

---

### Step 3: Create a Lambda Function with S3 Blueprint
1. Switch to the **Lambda tab**.  
2. Click on **Create Function** → Select **Use a blueprint**.  
3. Search for and select the blueprint **`s3-get-object` (Python)**.  
4. Keep the default execution role (first option: create a new role with basic Lambda permissions).  
5. ⚠️ Do **not** add the S3 trigger from Lambda. We will attach it later directly from the S3 bucket.

---

### Step 4: Modify the Lambda Code
In the generated blueprint code, update the **try block** to log the object details and its body:

```python
try:
    response = s3.get_object(Bucket=bucket, Key=key)
    print(response)
    print(response['Body'].read())
    # return response['ContentType']
````

Also, **comment out the first line of `lambda_handler`** if it references `event['Records'][0]` to avoid unnecessary issues.

---

### Step 5: Add S3 Event Notification

1. Go back to the **S3 Console** → Open your bucket.
2. Navigate to **Properties → Event notifications**.
3. Create a new event notification:

   * Event type: `PUT` (when an object is uploaded).
   * Destination: Your Lambda function.
4. Save changes.
   → Now your S3 bucket is linked with the Lambda function.

---

### Step 6: Test the Integration

1. Upload a file (e.g., `test.txt` or even an image) into your S3 bucket.
2. Check **CloudWatch Logs** for your Lambda function.

   * You should see the **object metadata** and the **file content** printed.

---

### Step 7: Fixing Permission Issues

If you see **Access Denied** errors:

1. Go to **S3 Bucket → Permissions**.
2. Scroll down to **Block public access (bucket settings)** → **Turn it off**.
3. Edit the **Bucket Policy** and add the following policy (update bucket name accordingly):

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

4. Save changes.
5. Re-upload your file to the bucket.

---

### Step 8: Verify Output

* Go to **CloudWatch Logs** again.
* You should now see the full **file content (body)** logged successfully.

---

With this, we completed an end-to-end workflow:

* **S3 bucket event → Lambda trigger → CloudWatch logs output.**
  This demonstrates how AWS Lambda can be used for **event-driven serverless processing**.

```


