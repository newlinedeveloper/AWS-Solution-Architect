# AWS Lambda

AWS Lambda is a serverless compute service that lets you run code without provisioning or managing servers. You pay only for the compute time you consume.

## Concurrency Limits

AWS Lambda has concurrency limits to control the number of instances of your function that can run simultaneously. The default concurrency limit per account is 1,000 concurrent executions.

### Example Use Case: Setting Concurrency Limits

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
    "github.com/constructs-go/constructs/v10"
)

type LambdaConcurrencyStackProps struct {
    awscdk.StackProps
}

func NewLambdaConcurrencyStack(scope constructs.Construct, id string, props *LambdaConcurrencyStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a Lambda function
    lambdaFunction := awslambda.NewFunction(stack, jsii.String("MyFunction"), &awslambda.FunctionProps{
        Runtime: awslambda.Runtime_NODEJS_14_X(),
        Handler: jsii.String("index.handler"),
        Code:    awslambda.Code_FromAsset(jsii.String("lambda")),
    })

    // Set concurrency limit
    lambdaFunction.AddAlias(jsii.String("live"), &awslambda.AliasOptions{
        ProvisionedConcurrentExecutions: jsii.Number(5),
    })

    // Output the function name
    awscdk.NewCfnOutput(stack, jsii.String("FunctionName"), &awscdk.CfnOutputProps{
        Value: lambdaFunction.FunctionName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewLambdaConcurrencyStack(app, "LambdaConcurrencyStack", &LambdaConcurrencyStackProps{
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

### Maximum Memory Allocation and Timeout Duration
AWS Lambda allows you to allocate memory from 128 MB to 10,240 MB and set a timeout duration from 1 second to 15 minutes.

**Example Use Case: Setting Memory Allocation and Timeout Duration**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
    "github.com/constructs-go/constructs/v10"
)

type LambdaMemoryTimeoutStackProps struct {
    awscdk.StackProps
}

func NewLambdaMemoryTimeoutStack(scope constructs.Construct, id string, props *LambdaMemoryTimeoutStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a Lambda function with specific memory and timeout settings
    lambdaFunction := awslambda.NewFunction(stack, jsii.String("MyFunction"), &awslambda.FunctionProps{
        Runtime: awslambda.Runtime_NODEJS_14_X(),
        Handler: jsii.String("index.handler"),
        Code:    awslambda.Code_FromAsset(jsii.String("lambda")),
        MemorySize: jsii.Number(1024),
        Timeout: awscdk.Duration_Seconds(jsii.Number(300)),
    })

    // Output the function name
    awscdk.NewCfnOutput(stack, jsii.String("FunctionName"), &awscdk.CfnOutputProps{
        Value: lambdaFunction.FunctionName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewLambdaMemoryTimeoutStack(app, "LambdaMemoryTimeoutStack", &LambdaMemoryTimeoutStackProps{
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

#### Lambda@Edge Computing
Lambda@Edge allows you to run Lambda functions at AWS edge locations in response to Amazon CloudFront events. This enables you to customize content delivery and reduce latency.

**Example Use Case: Creating a Lambda@Edge Function**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
    "github.com/aws/aws-cdk-go/awscdk/v2/awscloudfront"
    "github.com/constructs-go/constructs/v10"
)

type LambdaEdgeStackProps struct {
    awscdk.StackProps
}

func NewLambdaEdgeStack(scope constructs.Construct, id string, props *LambdaEdgeStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a Lambda function for Lambda@Edge
    lambdaFunction := awslambda.NewFunction(stack, jsii.String("MyEdgeFunction"), &awslambda.FunctionProps{
        Runtime: awslambda.Runtime_NODEJS_14_X(),
        Handler: jsii.String("index.handler"),
        Code:    awslambda.Code_FromAsset(jsii.String("lambda")),
    })

    // Create a CloudFront distribution with Lambda@Edge
    distribution := awscloudfront.NewDistribution(stack, jsii.String("MyDistribution"), &awscloudfront.DistributionProps{
        DefaultBehavior: &awscloudfront.BehaviorOptions{
            Origin: awscloudfront.NewOrigins_S3Origin(jsii.String("my-bucket")),
            EdgeLambdas: &[]*awscloudfront.EdgeLambda{
                {
                    FunctionVersion: lambdaFunction.CurrentVersion(),
                    EventType: awscloudfront.LambdaEdgeEventType_ORIGIN_REQUEST,
                },
            },
        },
    })

    // Output the distribution domain name
    awscdk.NewCfnOutput(stack, jsii.String("DistributionDomainName"), &awscdk.CfnOutputProps{
        Value: distribution.DistributionDomainName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewLambdaEdgeStack(app, "LambdaEdgeStack", &LambdaEdgeStackProps{
        awscdk.StackProps{
            Env: env(),
        },
    })

    app.Synth(nil)
}

func env() *awscdk.Environment {
    return &awscdk.Environment{
        Region: jsii.String("us-east-1"), // Lambda@Edge functions must be created in us-east-1
    }
}
```

#### Connecting Your Lambda Function to Your VPC
You can connect your Lambda function to your VPC to access resources such as RDS databases, ElastiCache clusters, or internal services.

**Example Use Case: Connecting a Lambda Function to a VPC**
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
    "github.com/constructs-go/constructs/v10"
)

type LambdaVPCStackProps struct {
    awscdk.StackProps
}

func NewLambdaVPCStack(scope constructs.Construct, id string, props *LambdaVPCStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create a Lambda function connected to the VPC
    lambdaFunction := awslambda.NewFunction(stack, jsii.String("MyFunction"), &awslambda.FunctionProps{
        Runtime: awslambda.Runtime_NODEJS_14_X(),
        Handler: jsii.String("index.handler"),
        Code:    awslambda.Code_FromAsset(jsii.String("lambda")),
        Vpc: vpc,
    })

    // Output the function name
    awscdk.NewCfnOutput(stack, jsii.String("FunctionName"), &awscdk.CfnOutputProps{
        Value: lambdaFunction.FunctionName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewLambdaVPCStack(app, "LambdaVPCStack", &LambdaVPCStackProps{
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

- AWS Lambda: A serverless compute service that lets you run code without provisioning or managing servers.
- Concurrency Limits: Control the number of instances of your function that can run simultaneously.
- Maximum Memory Allocation and Timeout Duration: Allocate memory from 128 MB to 10,240 MB and set a timeout duration from 1 second to 15 minutes.
- Lambda@Edge Computing: Run Lambda functions at AWS edge locations in response to Amazon CloudFront events.
- Connecting Your Lambda Function to Your VPC: Access resources such as RDS databases, ElastiCache clusters, or internal services.

