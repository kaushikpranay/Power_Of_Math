# Power Of Math

This is a simple serverless application built on AWS that calculates the power of a number (`base^exponent`).  
The result is stored in DynamoDB along with a timestamp, and returned as an API response.

---

## Project Overview

- Users send a request with two numbers: `base` and `exponent`.  
- The request goes through **API Gateway**.  
- A **Lambda function** calculates the result using Python.  
- The result and execution time are stored in a **DynamoDB** table.  
- The API returns the computed value in JSON format.

---

## Architecture

![Architecture Diagram](./Power_Of_Math.drawio.png)

---

## Technologies Used

- **AWS API Gateway** – to expose an HTTP endpoint  
- **AWS Lambda (Python 3.x)** – to process requests and perform calculations  
- **Amazon DynamoDB** – to store results and timestamps  
- **IAM Role** – to provide Lambda with DynamoDB access  

---

## IAM Policy

The Lambda function role includes DynamoDB permissions such as `PutItem`, `GetItem`, `UpdateItem`, and `Query`.  
Example policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:DeleteItem",
        "dynamodb:GetItem",
        "dynamodb:Scan",
        "dynamodb:Query",
        "dynamodb:UpdateItem"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:<account-id>:table/PowerOfMath"
    }
  ]
}
```

---

## Lambda Function

```python
import json
import math
import boto3
from time import gmtime, strftime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('PowerOfMath')
now = strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())

def lambda_handler(event, context):
    mathResult = math.pow(int(event['base']), int(event['exponent']))
    response = table.put_item(
        Item={
            'ID': str(mathResult),
            'LatestGreetingTime': now
        })
    return {
        'statusCode': 200,
        'body': json.dumps('Your result is ' + str(mathResult))
    }
```

---

## Setup Instructions

1. **Create DynamoDB Table**
   - Name: `PowerOfMath`  
   - Partition Key: `ID` (String)

2. **Deploy Lambda Function**
   - Runtime: Python 3.x  
   - Upload the function code  
   - Attach IAM role with DynamoDB permissions  

3. **Configure API Gateway**
   - Create a resource `/pow`  
   - Add `POST` method  
   - Integrate it with the Lambda function  

4. **Test the API**
   ```bash
   curl -X POST https://<api-id>.execute-api.<region>.amazonaws.com/prod/pow \
   -H "Content-Type: application/json" \
   -d '{"base": "2", "exponent": "10"}'
   ```

   Example Response:
   ```json
   {
     "statusCode": 200,
     "body": "\"Your result is 1024.0\""
   }
   ```

---

## Example DynamoDB Record

| ID    | LatestGreetingTime          |
|-------|-----------------------------|
| 1024  | Sun, 14 Sep 2025 17:00:00 +0000 |

---

## Possible Improvements

- Extend API to support more mathematical operations  
- Store request history (base, exponent, result) instead of just the result  
- Add a simple frontend interface to call the API  
