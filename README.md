## Introduction

MP3 to Podcast generates a custom, single-episode podcast from an MP3, allowing you to listen to one-off audio files in any podcast app.

It is implemented in two parts:

1. Static web client
2. Node.js package for deployment on [AWS Lambda](https://aws.amazon.com/lambda/)

The server-side works by simply translating query parameters sent from the static web page into a podcast RSS feed. You can see a live demo at http://schreifels.org/mp3-to-podcast/

## Deployment

### Client-side

The code in `client/` is static and can be deployed anywhere as-is.

### Server-side

On the server-side, MP3 to Podcast is intended to be an Amazon Web Services Lambda function.

Before you begin, generate the source code for the server:

```bash
cd server/
make
```

then go to the AWS Management Console and:

1. Go to AWS IAM.
2. Under Policies, select "Create Policy", then "Create Your Own Policy".
3. Call it `APIGatewayLambdaInvokePolicy`:

    ```
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Resource": [
            "*"
          ],
          "Action": [
            "lambda:InvokeFunction"
          ]
        }
      ]
    }
    ```

4. Under Roles, select "Create New Role", call it `APIGatewayLambdaInvokeRole`, and choose "Amazon API Gateway". Don't attach a policy.
5. Under Policies, select "Create Policy", then "Create Your Own Policy".
6. Call it `APIGatewayLambdaExecPolicy`:

    ```
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Action": [
            "logs:*"
          ],
          "Effect": "Allow",
          "Resource": "arn:aws:logs:*:*:*"
        }
      ]
    }
    ```

7. Under Roles, select "Create New Role", call it `APIGatewayLambdaExecRole`, and choose "AWS Lambda". Attach `APIGatewayLambdaExecPolicy`.
8. Go to AWS Lambda.
9. Create a new Lambda function with a blank blueprint.
10. Select the "Node.js 4.3" runtime.
11. Upload the zip file that was generated for you in `dist/`.
12. For your Lambda function handler role, choose `APIGatewayLambdaExecPolicy`.
13. Go to AWS API Gateway.
14. Create a new API Gateway by importing the Swagger file in `server/`.
15. In the Resources view, select "GET /", go to "Integration Request", click edit next to "Lambda Function: mp3ToPodcast", re-type the name of the Lambda function, press save, and accept the permissions prompt.
16. Under "Actions", click "Deploy API" and create a new stage called "prod" -- this will give you the API URL.
