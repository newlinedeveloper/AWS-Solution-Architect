# Amazon S3 Glacier

Amazon S3 Glacier is a secure, durable, and low-cost storage service for data archiving and long-term backup. It is designed to provide 99.999999999% durability and comprehensive security and compliance capabilities.

## Amazon S3 Glacier vs Amazon S3 Glacier Deep Archive

### Amazon S3 Glacier

- **Purpose**: Designed for long-term storage of infrequently accessed data.
- **Retrieval Times**: Offers three retrieval options: Expedited (1-5 minutes), Standard (3-5 hours), and Bulk (5-12 hours).
- **Use Case**: Ideal for data that needs to be accessed occasionally but requires long-term retention.

### Amazon S3 Glacier Deep Archive

- **Purpose**: Designed for long-term storage of rarely accessed data.
- **Retrieval Times**: Offers two retrieval options: Standard (12 hours) and Bulk (48 hours).
- **Use Case**: Ideal for data that is rarely accessed and requires long-term retention at the lowest cost.

## Example Use Case: Creating an S3 Bucket with Glacier and Glacier Deep Archive Storage Classes

### AWS CDK Code for Creating an S3 Bucket with Glacier and Glacier Deep Archive Storage Classes

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
    "github.com/constructs-go/constructs/v10"
)

type S3GlacierStackProps struct {
    awscdk.StackProps
}

func NewS3GlacierStack(scope constructs.Construct, id string, props *S3GlacierStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an S3 bucket
    bucket := awss3.NewBucket(stack, jsii.String("MyS3Bucket"), &awss3.BucketProps{
        Versioned: jsii.Bool(true),
        LifecycleRules: &[]*awss3.LifecycleRule{
            {
                Transitions: &[]*awss3.Transition{
                    {
                        StorageClass: awss3.StorageClass_GLACIER,
                        TransitionAfter: awscdk.Duration_Days(jsii.Number(30)),
                    },
                    {
                        StorageClass: awss3.StorageClass_DEEP_ARCHIVE,
                        TransitionAfter: awscdk.Duration_Days(jsii.Number(180)),
                    },
                },
            },
        },
    })

    // Output the bucket name
    awscdk.NewCfnOutput(stack, jsii.String("BucketName"), &awscdk.CfnOutputProps{
        Value: bucket.BucketName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewS3GlacierStack(app, "S3GlacierStack", &S3GlacierStackProps{
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
- Amazon S3 Glacier: A secure, durable, and low-cost storage service for data archiving and long-term backup with retrieval times ranging from minutes to hours.
- Amazon S3 Glacier Deep Archive: Designed for long-term storage of rarely accessed data with retrieval times ranging from 12 to 48 hours.
- Example Use Case: Creating an S3 bucket with lifecycle rules to transition objects to Glacier and Glacier Deep Archive storage classes using AWS CDK in Go.
