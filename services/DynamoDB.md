# Amazon DynamoDB

Amazon DynamoDB is a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability. It allows you to offload the administrative burdens of operating and scaling a distributed database, so you don't have to worry about hardware provisioning, setup and configuration, replication, software patching, or cluster scaling.

## Amazon DynamoDB Transactions

DynamoDB transactions provide atomic, consistent, isolated, and durable (ACID) transactions to help you maintain data integrity in your applications. Transactions enable you to perform multiple operations on multiple items as a single, all-or-nothing operation.

### Example Use Case: Creating a DynamoDB Table with Transactions

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsdynamodb"
    "github.com/constructs-go/constructs/v10"
)

type DynamoDBTransactionsStackProps struct {
    awscdk.StackProps
}

func NewDynamoDBTransactionsStack(scope constructs.Construct, id string, props *DynamoDBTransactionsStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a DynamoDB table with transactions enabled
    table := awsdynamodb.NewTable(stack, jsii.String("MyTable"), &awsdynamodb.TableProps{
        PartitionKey: &awsdynamodb.Attribute{
            Name: jsii.String("PK"),
            Type: awsdynamodb.AttributeType_STRING,
        },
        SortKey: &awsdynamodb.Attribute{
            Name: jsii.String("SK"),
            Type: awsdynamodb.AttributeType_STRING,
        },
        BillingMode: awsdynamodb.BillingMode_PAY_PER_REQUEST,
    })

    // Output the table name
    awscdk.NewCfnOutput(stack, jsii.String("TableName"), &awscdk.CfnOutputProps{
        Value: table.TableName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewDynamoDBTransactionsStack(app, "DynamoDBTransactionsStack", &DynamoDBTransactionsStackProps{
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
**AWS Lambda Integration with Amazon DynamoDB Streams**
DynamoDB Streams capture a time-ordered sequence of item-level changes in a DynamoDB table and store this information for up to 24 hours. You can integrate AWS Lambda with DynamoDB Streams to process these changes in real-time.

**Example Use Case: Creating a Lambda Function Triggered by DynamoDB Streams**
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsdynamodb"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambdaneventsources"
    "github.com/constructs-go/constructs/v10"
)

type DynamoDBStreamsStackProps struct {
    awscdk.StackProps
}

func NewDynamoDBStreamsStack(scope constructs.Construct, id string, props *DynamoDBStreamsStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a DynamoDB table with streams enabled
    table := awsdynamodb.NewTable(stack, jsii.String("MyTable"), &awsdynamodb.TableProps{
        PartitionKey: &awsdynamodb.Attribute{
            Name: jsii.String("PK"),
            Type: awsdynamodb.AttributeType_STRING,
        },
        SortKey: &awsdynamodb.Attribute{
            Name: jsii.String("SK"),
            Type: awsdynamodb.AttributeType_STRING,
        },
        BillingMode: awsdynamodb.BillingMode_PAY_PER_REQUEST,
        Stream:      awsdynamodb.StreamViewType_NEW_AND_OLD_IMAGES,
    })

    // Create a Lambda function
    lambdaFunction := awslambda.NewFunction(stack, jsii.String("MyFunction"), &awslambda.FunctionProps{
        Runtime: awslambda.Runtime_NODEJS_14_X(),
        Handler: jsii.String("index.handler"),
        Code:    awslambda.Code_FromAsset(jsii.String("lambda")),
    })

    // Add DynamoDB stream as an event source for the Lambda function
    lambdaFunction.AddEventSource(awslambdaneventsources.NewDynamoEventSource(table, &awslambdaneventsources.DynamoEventSourceProps{
        StartingPosition: awslambda.StartingPosition_LATEST,
    }))

    // Output the table name
    awscdk.NewCfnOutput(stack, jsii.String("TableName"), &awscdk.CfnOutputProps{
        Value: table.TableName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewDynamoDBStreamsStack(app, "DynamoDBStreamsStack", &DynamoDBStreamsStackProps{
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
**Amazon DynamoDB Replication**
DynamoDB Global Tables provide a fully managed solution for deploying a multi-region, multi-master database, without having to build and maintain your own replication solution.

Example Use Case: Creating a Global Table

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsdynamodb"
    "github.com/constructs-go/constructs/v10"
)

type DynamoDBGlobalTableStackProps struct {
    awscdk.StackProps
}

func NewDynamoDBGlobalTableStack(scope constructs.Construct, id string, props *DynamoDBGlobalTableStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a DynamoDB table
    table := awsdynamodb.NewTable(stack, jsii.String("MyTable"), &awsdynamodb.TableProps{
        PartitionKey: &awsdynamodb.Attribute{
            Name: jsii.String("PK"),
            Type: awsdynamodb.AttributeType_STRING,
        },
        SortKey: &awsdynamodb.Attribute{
            Name: jsii.String("SK"),
            Type: awsdynamodb.AttributeType_STRING,
        },
        BillingMode: awsdynamodb.BillingMode_PAY_PER_REQUEST,
    })

    // Enable global table replication
    table.AddGlobalSecondaryIndex(&awsdynamodb.GlobalSecondaryIndexProps{
        IndexName: jsii.String("GlobalIndex"),
        PartitionKey: &awsdynamodb.Attribute{
            Name: jsii.String("GPK"),
            Type: awsdynamodb.AttributeType_STRING,
        },
        SortKey: &awsdynamodb.Attribute{
            Name: jsii.String("GSK"),
            Type: awsdynamodb.AttributeType_STRING,
        },
    })

    // Output the table name
    awscdk.NewCfnOutput(stack, jsii.String("TableName"), &awscdk.CfnOutputProps{
        Value: table.TableName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewDynamoDBGlobalTableStack(app, "DynamoDBGlobalTableStack", &DynamoDBGlobalTableStackProps{
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
**Caching with DynamoDB DAX**
DynamoDB Accelerator (DAX) is a fully managed, highly available, in-memory caching service for DynamoDB that delivers up to a 10x performance improvement.

**Example Use Case: Creating a DAX Cluster**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsdax"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type DynamoDBDAXStackProps struct {
    awscdk.StackProps
}

func NewDynamoDBDAXStack(scope constructs.Construct, id string, props *DynamoDBDAXStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create a DAX cluster
    daxCluster := awsdax.NewCfnCluster(stack, jsii.String("MyDAXCluster"), &awsdax.CfnClusterProps{
        IamRoleArn: jsii.String("arn:aws:iam::123456789012:role/DAXServiceRole"),
        NodeType:   jsii.String("dax.r4.large"),
        ReplicationFactor: jsii.Number(3),
        SubnetGroupName: jsii.String("MySubnetGroup"),
    })

    // Output the DAX cluster endpoint
    awscdk.NewCfnOutput(stack, jsii.String("DAXClusterEndpoint"), &awscdk.CfnOutputProps{
        Value: daxCluster.AttrClusterDiscoveryEndpoint(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewDynamoDBDAXStack(app, "DynamoDBDAXStack", &DynamoDBDAXStackProps{
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

- Amazon DynamoDB: A fully managed NoSQL database service that provides fast and predictable performance with seamless scalability.
- DynamoDB Transactions: Enable ACID transactions to maintain data integrity.
- AWS Lambda Integration with DynamoDB Streams: Process real-time changes in DynamoDB tables using Lambda functions.
- DynamoDB Replication: Use Global Tables for multi-region, multi-master replication.
- Caching with DynamoDB DAX: Improve performance with in-memory caching.

