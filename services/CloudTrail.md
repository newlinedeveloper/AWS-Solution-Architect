# AWS CloudTrail

AWS CloudTrail is a service that enables governance, compliance, and operational and risk auditing of your AWS account. With CloudTrail, you can log, continuously monitor, and retain account activity related to actions across your AWS infrastructure.

## What’s Not Monitored By Default in CloudTrail

By default, AWS CloudTrail logs management events, which include operations that create, modify, or delete AWS resources. However, some activities are not monitored by default:

- **Data Events**: These include operations performed on or within a resource, such as S3 object-level operations (e.g., GetObject, PutObject) and Lambda function invocations.
- **Insights Events**: These provide insights into unusual operational activity in your AWS account.

### How To Start Monitoring Data Events and Insights Events

To start monitoring data events and insights events, you need to configure your CloudTrail trail to log these events.

### Example: Enabling Data Events and Insights Events Using AWS CDK (Go)

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3_notifications"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
    "github.com/aws/aws-cdk-go/awscdk/v2/awscloudtrail"
    "github.com/aws/constructs-go/constructs/v10"
)

type CloudTrailStackProps struct {
    awscdk.StackProps
}

func NewCloudTrailStack(scope constructs.Construct, id string, props *CloudTrailStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an S3 bucket for CloudTrail logs
    bucket := awss3.NewBucket(stack, jsii.String("CloudTrailBucket"), &awss3.BucketProps{
        Versioned: jsii.Bool(true),
    })

    // Create a CloudTrail trail
    trail := awscloudtrail.NewTrail(stack, jsii.String("MyTrail"), &awscloudtrail.TrailProps{
        Bucket: bucket,
        IsMultiRegionTrail: jsii.Bool(true),
        IncludeGlobalServiceEvents: jsii.Bool(true),
        ManagementEvents: awscloudtrail.ReadWriteType_ALL,
        InsightTypes: &[]awscloudtrail.InsightType{
            awscloudtrail.InsightType_API_CALL_RATE,
            awscloudtrail.InsightType_API_ERROR_RATE,
        },
    })

    // Add an S3 data event selector
    trail.AddS3EventSelector(&[]*string{
        jsii.String("arn:aws:s3:::my-bucket/"),
    }, &awscloudtrail.AddEventSelectorOptions{
        ReadWriteType: awscloudtrail.ReadWriteType_ALL,
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewCloudTrailStack(app, "CloudTrailStack", &CloudTrailStackProps{
        awscdk.StackProps{
            Env: env(),
        },
    })

    app.Synth(nil)
}

func env() *awscdk.Environment {
    return &awscdk.Environment{
        Region: jsii.String("us-west-2"),
    }
}

```

Receiving CloudTrail Logs from Multiple Accounts and Sharing Logs To Other Accounts

To centralize CloudTrail logs from multiple accounts, you can create a CloudTrail trail in each account and configure them to deliver logs to a central S3 bucket in a designated account. You can also share these logs with other accounts.

Steps
- Create a Central S3 Bucket: Create an S3 bucket in the designated account to store CloudTrail logs.
- Configure CloudTrail in Each Account: Configure CloudTrail in each account to deliver logs to the central S3 bucket.
- Set Bucket Policies: Set bucket policies to allow CloudTrail from other accounts to write logs to the central S3 bucket.
- Share Logs: Share the logs with other accounts by granting appropriate permissions.

**Example: Centralizing CloudTrail Logs Using AWS CDK (Go)**

Central Account: Create Central S3 Bucket and Set Bucket Policies
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
    "github.com/aws/constructs-go/constructs/v10"
)

type CentralBucketStackProps struct {
    awscdk.StackProps
}

func NewCentralBucketStack(scope constructs.Construct, id string, props *CentralBucketStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a central S3 bucket for CloudTrail logs
    bucket := awss3.NewBucket(stack, jsii.String("CentralCloudTrailBucket"), &awss3.BucketProps{
        Versioned: jsii.Bool(true),
    })

    // Add bucket policy to allow CloudTrail from other accounts to write logs
    bucket.AddToResourcePolicy(awss3.NewPolicyStatement(&awss3.PolicyStatementProps{
        Actions: &[]*string{
            jsii.String("s3:PutObject"),
        },
        Resources: &[]*string{
            bucket.ArnForObjects(jsii.String("AWSLogs/*")),
        },
        Principals: &[]awss3.IPrincipal{
            awss3.NewAccountPrincipal(jsii.String("123456789012")), // Replace with the account ID of the other account
        },
    }))

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewCentralBucketStack(app, "CentralBucketStack", &CentralBucketStackProps{
        awscdk.StackProps{
            Env: env(),
        },
    })

    app.Synth(nil)
}

func env() *awscdk.Environment {
    return &awscdk.Environment{
        Region: jsii.String("us-west-2"),
    }
}
```

**Other Accounts: Configure CloudTrail to Deliver Logs to Central S3 Bucket**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
    "github.com/aws/aws-cdk-go/awscdk/v2/awscloudtrail"
    "github.com/aws/constructs-go/constructs/v10"
)

type CloudTrailStackProps struct {
    awscdk.StackProps
}

func NewCloudTrailStack(scope constructs.Construct, id string, props *CloudTrailStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Reference the central S3 bucket
    bucket := awss3.Bucket_FromBucketArn(stack, jsii.String("CentralBucket"), jsii.String("arn:aws:s3:::central-cloudtrail-bucket"))

    // Create a CloudTrail trail
    trail := awscloudtrail.NewTrail(stack, jsii.String("MyTrail"), &awscloudtrail.TrailProps{
        Bucket: bucket,
        IsMultiRegionTrail: jsii.Bool(true),
        IncludeGlobalServiceEvents: jsii.Bool(true),
        ManagementEvents: awscloudtrail.ReadWriteType_ALL,
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewCloudTrailStack(app, "CloudTrailStack", &CloudTrailStackProps{
        awscdk.StackProps{
            Env: env(),
        },
    })

    app.Synth(nil)
}

func env() *awscdk.Environment {
    return &awscdk.Environment{
        Region: jsii.String("us-west-2"),
    }
}

```

- Management Events: Include operations that create, modify, or delete AWS resources, IAM actions, and AWS service configuration changes. Logged by default.
- Global Service Events: Include operations performed on global services such as IAM, CloudFront, and Route 53. Logged by default.
- API Activity: Include API calls made to AWS services via the AWS Management Console, SDKs, CLI, and service-to-service calls. Logged by default.

By default, AWS CloudTrail provides comprehensive visibility into the management operations, global service events, and API activity in your AWS account, helping you ensure governance, compliance, and operational auditing.

```

