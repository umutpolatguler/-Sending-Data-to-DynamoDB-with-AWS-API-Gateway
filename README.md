# Sending Data to DynamoDB with AWS API Gateway

This documentation explains how to send data using AWS API Gateway, Lambda and DynamoDB. Includes testing the API using Postman.

## Step 1: Create a DynamoDB Table

1. In the AWS Management Console, navigate to the **DynamoDB** service.
2. Click the **Create Table** button.
3. **Table Name**: `Users` (table name)
4. **Primary Key**: 
   - **Partition Key**: `userId` (Type: `String`)
   - **Sort Key**: `creationDate` (Type: `String`)

## Step 2: Creating a Lambda Function

1. In the AWS Management Console, navigate to the **Lambda** service.
2. Click the **Create Function** button.
3. **Function Name**: `lambdaHandler`
4. **Runtime**: `Python 3.12` or more recent version.
5. **Execution Role**: Select a role with the permission to write to DynamoDB (e.g. `AmazonDynamoDBFullAccess`.)
6. Edit the contents of the lambda function as follows:
   ```python
   import json
   import boto3

   def lambda_handler(event, context):

        if isinstance(event['body'], str):
            item = json.loads(event['body'])
        else:
            item = event['body']

        if 'userId' not in item:
            return {
                'statusCode': 400,
                'body': json.dumps('Validation Error: Missing userId in the item')
            }

        dynamodb = boto3.resource('dynamodb')
        table = dynamodb.Table('Users')
        table.put_item(Item=item)

        return {
            'statusCode': 200,
            'body': json.dumps('Data added to the DynamoDB Table')
        }
    ```

## Step 3: Creating a RESTful API with API Gateway

1. In the AWS Management Console, navigate to the **API Gateway** service.
2. Click the **Create API** button.
3. Select **REST API**.
4. Select **New API**.
5. **API Name**: `UserAPI`
6. **Endpoint Type**: `Regional` or `Edge-Optimized` depending on the need.
7. Click the **Create Resource** button.
8. **Resource Name**: `users`
9. **Resource Path**: `/users`
10. Add the **POST** method.
11. Select `Lambda Function` as the **Integration type**.
12. Select the lambda function (`lambdaHandler`).

## Step 4: Connecting the Lambda Function to the API Gateway

1. Go to **Integration Request** for the POST method.
2. Under **Mapping Templates** add a template in `application/json` format:
```json
{
    "body": $input.json('$')
}
```
3. Click **Deploy API** from the **Actions** menu. Create a new **Deployment Stage** (for example `dev`).

## Step 5: Testing the API with Postman

1. Open the Postman application.
2. Create a new `POST` request.
3. Enter the URL: `https://<api-id>.execute-api.<region>.amazonaws.com/dev/users` (It can also be copied from the API dashboard.)
4. On the **Body** tab, select `raw` and `JSON` format.
5. Enter a JSON data in this format:
    ```json
    {
        "userId": "FireFrost0665",
        "creationDate": "1996-08-16"
    }
    ```
6. Click the **Send** button and check the status code.

It should give the status code `200` and the message `Data added to the DynamoDB Table`.
