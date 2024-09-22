# Amazon Simple Notification Service (SNS)

Amazon Simple Notification Service (SNS) is a fully managed messaging service for both application-to-application (A2A) and application-to-person (A2P) communication. It enables you to decouple microservices, distributed systems, and serverless applications.

## Features

- **Fully Managed**: No need to manage infrastructure; AWS handles the underlying infrastructure.
- **Scalable**: Automatically scales to handle any amount of messages.
- **Reliable**: Ensures delivery of messages with high durability.
- **Secure**: Provides encryption at rest and in transit.
- **Flexible**: Supports multiple protocols (HTTP/S, Email, SMS, Lambda, SQS).

## Amazon SNS Message Filtering

Message filtering allows subscribers to receive only the messages that are relevant to them. This is achieved by defining filter policies on the subscription.

### Example: Creating an SNS Topic and Subscription with Message Filtering Using AWS CDK (Go)

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns_subscriptions"
    "github.com/aws/constructs-go/constructs/v10"
)

type SnsTopicStackProps struct {
    awscdk.StackProps
}

func NewSnsTopicStack(scope constructs.Construct, id string, props *SnsTopicStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an SNS topic
    topic := awssns.NewTopic(stack, jsii.String("MyTopic"), &awssns.TopicProps{
        TopicName: jsii.String("my-topic"),
    })

    // Create an SQS queue
    queue := awssqs.NewQueue(stack, jsii.String("MyQueue"), &awssqs.QueueProps{
        QueueName: jsii.String("my-queue"),
    })

    // Subscribe the queue to the topic with a filter policy
    topic.AddSubscription(awssns_subscriptions.NewSqsSubscription(queue, &awssns_subscriptions.SqsSubscriptionProps{
        FilterPolicy: &map[string]awssns.SubscriptionFilter{
            "eventType": awssns.SubscriptionFilter_StringFilter(&awssns.StringConditions{
                Allowlist: &[]*string{jsii.String("order_placed")},
            }),
        },
    }))

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewSnsTopicStack(app, "SnsTopicStack", &SnsTopicStackProps{
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
#### Amazon SNS Topic Types, Message Ordering, and Deduplication

Amazon SNS supports two types of topics: Standard and FIFO (First-In-First-Out). FIFO topics ensure that messages are delivered in the order they are sent and are not duplicated.

**Example: Creating a FIFO SNS Topic Using AWS CDK (Go)**
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns"
    "github.com/aws/constructs-go/constructs/v10"
)

type SnsFifoTopicStackProps struct {
    awscdk.StackProps
}

func NewSnsFifoTopicStack(scope constructs.Construct, id string, props *SnsFifoTopicStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a FIFO SNS topic
    fifoTopic := awssns.NewTopic(stack, jsii.String("MyFifoTopic"), &awssns.TopicProps{
        TopicName: jsii.String("my-fifo-topic.fifo"),
        Fifo:      jsii.Bool(true),
        ContentBasedDeduplication: jsii.Bool(true),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewSnsFifoTopicStack(app, "SnsFifoTopicStack", &SnsFifoTopicStackProps{
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

#### Invoke Lambda Functions Using SNS Subscription

Amazon SNS can invoke AWS Lambda functions when a message is published to a topic. This allows you to process messages in real-time using serverless compute.

**Example: Creating an SNS Topic and Lambda Subscription Using AWS CDK (Go)**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns_subscriptions"
    "github.com/aws/constructs-go/constructs/v10"
)

type SnsLambdaStackProps struct {
    awscdk.StackProps
}

func NewSnsLambdaStack(scope constructs.Construct, id string, props *SnsLambdaStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an SNS topic
    topic := awssns.NewTopic(stack, jsii.String("MyTopic"), &awssns.TopicProps{
        TopicName: jsii.String("my-topic"),
    })

    // Create a Lambda function
    lambdaFn := awslambda.NewFunction(stack, jsii.String("MyFunction"), &awslambda.FunctionProps{
        Runtime: awslambda.Runtime_GO_1_X(),
        Handler: jsii.String("main"),
        Code:    awslambda.Code_FromAsset(jsii.String("path/to/your/code")),
    })

    // Subscribe the Lambda function to the topic
    topic.AddSubscription(awssns_subscriptions.NewLambdaSubscription(lambdaFn))

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewSnsLambdaStack(app, "SnsLambdaStack", &SnsLambdaStackProps{
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

# Amazon SNS Message Filtering

Amazon SNS message filtering allows subscribers to receive only the messages that are relevant to them. This is achieved by defining filter policies on the subscription.

## How SNS Message Filtering Works

1. **Filter Policies**: Each subscription can have a filter policy that specifies which messages it should receive based on message attributes.
2. **Message Attributes**: Messages sent to the SNS topic must include attributes that the filter policies can evaluate.
3. **Multiple Subscriptions**: You can have multiple subscriptions with different filter conditions, allowing each subscriber to receive only the messages that match its filter policy.

## Example: Creating an SNS Topic with Multiple SQS Subscriptions Using AWS CDK (Go)

### Steps

1. **Create an SNS Topic**.
2. **Create SQS Queues**.
3. **Subscribe the SQS Queues to the SNS Topic with Filter Policies**.

### Example Code

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns_subscriptions"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssqs"
    "github.com/aws/constructs-go/constructs/v10"
)

type SnsTopicStackProps struct {
    awscdk.StackProps
}

func NewSnsTopicStack(scope constructs.Construct, id string, props *SnsTopicStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an SNS topic
    topic := awssns.NewTopic(stack, jsii.String("MyTopic"), &awssns.TopicProps{
        TopicName: jsii.String("my-topic"),
    })

    // Create SQS queues
    queue1 := awssqs.NewQueue(stack, jsii.String("MyQueue1"), &awssqs.QueueProps{
        QueueName: jsii.String("my-queue-1"),
    })

    queue2 := awssqs.NewQueue(stack, jsii.String("MyQueue2"), &awssqs.QueueProps{
        QueueName: jsii.String("my-queue-2"),
    })

    // Subscribe the queues to the topic with filter policies
    topic.AddSubscription(awssns_subscriptions.NewSqsSubscription(queue1, &awssns_subscriptions.SqsSubscriptionProps{
        FilterPolicy: &map[string]awssns.SubscriptionFilter{
            "eventType": awssns.SubscriptionFilter_StringFilter(&awssns.StringConditions{
                Allowlist: &[]*string{jsii.String("order_placed")},
            }),
        },
    }))

    topic.AddSubscription(awssns_subscriptions.NewSqsSubscription(queue2, &awssns_subscriptions.SqsSubscriptionProps{
        FilterPolicy: &map[string]awssns.SubscriptionFilter{
            "eventType": awssns.SubscriptionFilter_StringFilter(&awssns.StringConditions{
                Allowlist: &[]*string{jsii.String("order_cancelled")},
            }),
        },
    }))

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewSnsTopicStack(app, "SnsTopicStack", &SnsTopicStackProps{
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

**Example: Sending a Message with Attributes Using Python**

```
import boto3
from botocore.exceptions import NoCredentialsError, PartialCredentialsError

def send_message(topic_arn, message, attributes):
    try:
        sns_client = boto3.client('sns', region_name='us-west-2')
        response = sns_client.publish(
            TopicArn=topic_arn,
            Message=message,
            MessageAttributes=attributes
        )
        print(f'Message ID: {response["MessageId"]}')
    except NoCredentialsError:
        print("Credentials not available")
    except PartialCredentialsError:
        print("Incomplete credentials provided")
    except Exception as e:
        print(f"An error occurred: {e}")

if __name__ == "__main__":
    topic_arn = 'arn:aws:sns:us-west-2:123456789012:my-topic'
    
    # Send a message for order placed
    send_message(
        topic_arn,
        'This is a test message for order placed',
        {
            'eventType': {
                'DataType': 'String',
                'StringValue': 'order_placed'
            }
        }
    )
    
    # Send a message for order cancelled
    send_message(
        topic_arn,
        'This is a test message for order cancelled',
        {
            'eventType': {
                'DataType': 'String',
                'StringValue': 'order_cancelled'
            }
        }
    )

```


