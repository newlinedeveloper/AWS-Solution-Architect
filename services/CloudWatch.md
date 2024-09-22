# Amazon CloudWatch

Amazon CloudWatch is a monitoring and observability service that provides data and actionable insights to monitor your applications, respond to system-wide performance changes, optimize resource utilization, and get a unified view of operational health.

## Monitoring Additional Metrics with the CloudWatch Agent

The CloudWatch Agent allows you to collect additional system-level metrics and custom application metrics from your instances. It can be installed on both EC2 instances and on-premises servers.

### Steps to Install and Configure the CloudWatch Agent

1. **Install the CloudWatch Agent**:
   - Download and install the CloudWatch Agent on your instance.

2. **Configure the CloudWatch Agent**:
   - Create a configuration file to specify which metrics to collect.

3. **Start the CloudWatch Agent**:
   - Start the CloudWatch Agent to begin collecting and sending metrics to CloudWatch.

### Example: Installing and Configuring the CloudWatch Agent on an EC2 Instance

```bash
# Download the CloudWatch Agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm

# Install the CloudWatch Agent
sudo rpm -U ./amazon-cloudwatch-agent.rpm

# Create a configuration file (example: /opt/aws/amazon-cloudwatch-agent/bin/config.json)
sudo vi /opt/aws/amazon-cloudwatch-agent/bin/config.json

# Example configuration file content
{
  "metrics": {
    "namespace": "MyCustomNamespace",
    "metrics_collected": {
      "cpu": {
        "measurement": [
          {"name": "cpu_usage_idle", "rename": "CPU_IDLE", "unit": "Percent"}
        ],
        "totalcpu": true,
        "resources": ["*"]
      },
      "mem": {
        "measurement": [
          {"name": "mem_used_percent", "rename": "MEMORY_USED_PERCENT", "unit": "Percent"}
        ]
      }
    }
  }
}

# Start the CloudWatch Agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

```
#### CloudWatch Alarms for Triggering Actions

CloudWatch Alarms allow you to monitor CloudWatch metrics and automatically perform actions based on predefined thresholds. You can use alarms to send notifications or trigger automated actions.

**Example: Creating a CloudWatch Alarm Using AWS CDK (Go)**
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awscloudwatch"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns_subscriptions"
    "github.com/constructs-go/constructs/v10"
)

type CloudWatchAlarmStackProps struct {
    awscdk.StackProps
}

func NewCloudWatchAlarmStack(scope constructs.Construct, id string, props *CloudWatchAlarmStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an SNS topic for alarm notifications
    topic := awssns.NewTopic(stack, jsii.String("AlarmTopic"), &awssns.TopicProps{
        TopicName: jsii.String("alarm-topic"),
    })

    // Subscribe an email endpoint to the SNS topic
    topic.AddSubscription(awssns_subscriptions.NewEmailSubscription(jsii.String("example@example.com")))

    // Create a CloudWatch alarm
    alarm := awscloudwatch.NewAlarm(stack, jsii.String("HighCPUAlarm"), &awscloudwatch.AlarmProps{
        Metric: awscloudwatch.Metric{
            Namespace:  jsii.String("AWS/EC2"),
            MetricName: jsii.String("CPUUtilization"),
            DimensionsMap: map[string]*string{
                "InstanceId": jsii.String("i-1234567890abcdef0"),
            },
        },
        Threshold: jsii.Number(80),
        EvaluationPeriods: jsii.Number(1),
        ComparisonOperator: awscloudwatch.ComparisonOperator_GREATER_THAN_OR_EQUAL_TO_THRESHOLD,
    })

    // Add an action to the alarm to send a notification to the SNS topic
    alarm.AddAlarmAction(awscloudwatch.NewSnsAction(topic))

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewCloudWatchAlarmStack(app, "CloudWatchAlarmStack", &CloudWatchAlarmStackProps{
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
#### CloudWatch Events (Amazon EventBridge) for Specific Events and Recurring Tasks
Amazon EventBridge (formerly CloudWatch Events) allows you to create rules that match incoming events and route them to targets for processing. You can use EventBridge to respond to specific events or schedule recurring tasks.

**Example: Creating an EventBridge Rule for a Specific Event Using AWS CDK (Go)**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsevents"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsevents_targets"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
    "github.com/constructs-go/constructs/v10"
)

type EventBridgeStackProps struct {
    awscdk.StackProps
}

func NewEventBridgeStack(scope constructs.Construct, id string, props *EventBridgeStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a Lambda function to handle events
    lambdaFn := awslambda.NewFunction(stack, jsii.String("MyFunction"), &awslambda.FunctionProps{
        Runtime: awslambda.Runtime_GO_1_X(),
        Handler: jsii.String("main"),
        Code:    awslambda.Code_FromAsset(jsii.String("path/to/your/code")),
    })

    // Create an EventBridge rule for a specific event
    rule := awsevents.NewRule(stack, jsii.String("MyRule"), &awsevents.RuleProps{
        EventPattern: &awsevents.EventPattern{
            Source: &[]*string{jsii.String("aws.ec2")},
            DetailType: &[]*string{jsii.String("EC2 Instance State-change Notification")},
            Detail: &map[string]interface{}{
                "state": []string{"running"},
            },
        },
    })

    // Add the Lambda function as a target for the rule
    rule.AddTarget(awsevents_targets.NewLambdaFunction(lambdaFn))

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewEventBridgeStack(app, "EventBridgeStack", &EventBridgeStackProps{
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
**Example: Creating an EventBridge Rule for a Recurring Task Using AWS CDK (Go)**
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsevents"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsevents_targets"
    "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
    "github.com/constructs-go/constructs/v10"
)

type RecurringTaskStackProps struct {
    awscdk.StackProps
}

func NewRecurringTaskStack(scope constructs.Construct, id string, props *RecurringTaskStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a Lambda function to handle the recurring task
    lambdaFn := awslambda.NewFunction(stack, jsii.String("RecurringTaskFunction"), &awslambda.FunctionProps{
        Runtime: awslambda.Runtime_GO_1_X(),
        Handler: jsii.String("main"),
        Code:    awslambda.Code_FromAsset(jsii.String("path/to/your/code")),
    })

    // Create an EventBridge rule for a recurring task
    rule := awsevents.NewRule(stack, jsii.String("RecurringTaskRule"), &awsevents.RuleProps{
        Schedule: awsevents.Schedule_Rate(awscdk.Duration_Minutes(jsii.Number(5))),
    })

    // Add the Lambda function as a target for the rule
    rule.AddTarget(awsevents_targets.NewLambdaFunction(lambdaFn))

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewRecurringTaskStack(app, "RecurringTaskStack", &RecurringTaskStackProps{
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

- Amazon CloudWatch: A monitoring and observability service that provides data and actionable insights.
- Monitoring Additional Metrics: Use the CloudWatch Agent to collect additional system-level and custom application metrics.
- CloudWatch Alarms: Monitor metrics and automatically perform actions based on predefined thresholds.
