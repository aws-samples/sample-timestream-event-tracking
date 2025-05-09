# Timestream Event Tracking

## Overview

This CloudFormation template provisions a serverless event tracking system. The service allows clients to log "start" and "stop" events and later query the total elapsed time for a specific `event_id`. It uses AWS Lambda for compute, Amazon Timestream for time-series storage, and API Gateway to expose RESTful endpoints. All components are deployed within a VPC and use IAM for access control.

## Template Resources

**API Gateway**

POST `/event` – Logs a start or stop event

GET `/query` – Returns total elapsed time for a given event ID

**Lambda Functions**

`EventTrackingLambda`: Accepts and validates incoming events, writes to Timestream.

`EventQueryLambda`: Queries Timestream for start/stop events and calculates total duration.

**Amazon Timestream**

Stores event records with millisecond timestamps, organized by event ID and type.

**Amazon SQS (Dead Letter Queue)**

Captures failed Lambda invocations for diagnostics and retry handling.

**IAM Role**

Grants the Lambda functions least-privilege access to Timestream, SQS, logs, and VPC.

## Deployment Instructions

**Pre-requisites**
- An AWS account with permissions to deploy CloudFormation stacks.
- Pre-existing VPC with private subnets and a security group.
- AWS CLI or Management Console access.

**Deploy the Template**

- **Using the CLI**

   ```bash
   aws cloudformation deploy \
     --template-file logging.yaml \
     --stack-name event-tracking-service \
     --capabilities CAPABILITY_NAMED_IAM
    ```

Replace `logging.yaml` with the file path of your CloudFormation template. 

- **Using AWS Console**

1. Go to the [AWS CloudFormation console](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1), and click Create Stack.
2. Provide a stack name.
3. Select "Upload a template file" and upload the logging.yaml file from this repository.
4. On the Options page, click Next.
5. On the Review page, review the details and select the checkbox acknowledging that your template has capabilities to create AWS IAM resources. When finished, click Create. It may take several minutes for the stack to finish creating resources.


**Outputs**

- `LogApiEndpoint:` Invoke to log envents (POST)
- `QueryApiEndpoint:` Invoke to retrieve elapsed time (GET)

## Example Usage

**Log an Event**

```curl -X POST https://<api-id>.execute-api.<region>.amazonaws.com/prod/event \
  -H "Content-Type: application/json" \
  --data '{"event_id": "you-event-id", "event_type": "start"}'
```

**Query Elapsed Time**

```
curl "https://<api-id>.execute-api.<region>.amazonaws.com/prod/query?event_id=<your-event-id>"
```

## Cleanup

To remove the AWS CloudFormation Stack along with the AWS Resources created, navigate to the CloudFormation console and delete the stack. 

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

