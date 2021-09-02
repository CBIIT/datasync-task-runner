# datasync-task-runner

### sync-halo-images.yaml

This is a CloudFormation template for a serverless application which executes a DataSync task on a daily basis. Each task execution is configured to copy a folder (/YYYY-MM-DD) generated during the previous day. Notifications are delivered upon task execution status changes (eg: ERROR or SUCCESS events).

To create the application's stack, provide the following parameters:
- **SourceLocationId** - A DataSync source location from which files will be copied
- **DestinationLocationId** - A DataSync destination  location to which files will be copied
- **NotificationEmail** - An email address which will receieve notifications about DataSync task execution. Note that additional subscriptions can be added manually to the SNS topic.

The following resources are created:
- A **DataSync task** which syncs user-provided source and destination locations
  - A **CloudWatch log group** for the DataSync task which collects task execution log events
  - A **CloudWatch logs resource policy** which allows the DataSync service to log events to the CloudWatch log group
- A **Lambda function** which executes the DataSync task with appropriate filters
  - A **CloudWatch log group** for the Lambda function
  - An **IAM Role** for the Lambda function which allows it to create log streams/events in the specified log group, and to execute the DataSync task
  - An **EventBridge rule** which invokes the Lambda function on a daily basis
  - A **resource-based policy** which allows the EventBridge rule to invoke the Lambda function
- An **SNS Topic** which delivers notifications about DataSync task execution status (eg: ERROR or SUCCESS)
- A **Lambda function** which publishes messages to the SNS topic
  - A **CloudWatch log group** for the Lambda function
  - An **IAM Role** for the Lambda function which allows it to create log streams/events in the specified log group, retrieve the DataSync task's logs and status, and to publish messages to the SNS topic
  - An **EventBridge rule** which invokes the Lambda function whenever the DataSync task's execution status changes
  - A **resource-based policy** which allows the EventBridge rule to invoke the Lambda function
