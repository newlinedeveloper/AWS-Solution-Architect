# Amazon EC2 Auto Scaling

Amazon EC2 Auto Scaling helps you maintain application availability and allows you to automatically add or remove EC2 instances according to conditions you define. It ensures that you have the right number of EC2 instances available to handle the load for your application.

## Horizontal Scaling and Vertical Scaling

### Horizontal Scaling

- **Purpose**: Involves adding more instances to your Auto Scaling group.
- **Use Case**: Suitable for applications that can distribute load across multiple instances.

### Vertical Scaling

- **Purpose**: Involves increasing the size (e.g., CPU, memory) of an existing instance.
- **Use Case**: Suitable for applications that require more resources but cannot distribute load across multiple instances.

## Components of an AWS EC2 Auto Scaling Group

- **Launch Configuration/Template**: Defines the instance configuration, such as AMI, instance type, key pair, security groups, and block device mappings.
- **Auto Scaling Group**: Manages a collection of EC2 instances and includes configuration options such as minimum, maximum, and desired capacity.
- **Scaling Policies**: Define how to scale the Auto Scaling group in response to changes in demand.

## Types of EC2 Auto Scaling Policies

### Target Tracking Scaling

- **Purpose**: Adjusts the number of instances to maintain a target value for a specific metric.
- **Use Case**: Suitable for maintaining a specific performance metric, such as CPU utilization.

### Step Scaling

- **Purpose**: Adjusts the number of instances in response to changes in a metric, with predefined steps.
- **Use Case**: Suitable for applications with predictable traffic patterns.

### Simple Scaling

- **Purpose**: Adjusts the number of instances based on a single scaling adjustment.
- **Use Case**: Suitable for basic scaling needs.

## EC2 Auto Scaling Lifecycle Hooks

Lifecycle hooks enable you to perform custom actions as the Auto Scaling group launches or terminates instances.

### Example Use Case: Configuring Lifecycle Hooks

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns"
    "github.com/aws/aws-cdk-go/awscdk/v2/awssns_subscriptions"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsautoscaling"
    "github.com/constructs-go/constructs/v10"
)

type AutoScalingLifecycleHooksStackProps struct {
    awscdk.StackProps
}

func NewAutoScalingLifecycleHooksStack(scope constructs.Construct, id string, props *AutoScalingLifecycleHooksStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an Auto Scaling group
    asg := awsautoscaling.NewAutoScalingGroup(stack, jsii.String("MyAutoScalingGroup"), &awsautoscaling.AutoScalingGroupProps{
        Vpc: vpc,
        InstanceType: awsec2.InstanceType_Of(awsec2.InstanceClass_BURSTABLE2, awsec2.InstanceSize_MICRO),
        MachineImage: awsec2.NewAmazonLinuxImage(&awsec2.AmazonLinuxImageProps{}),
        MinCapacity: jsii.Number(1),
        MaxCapacity: jsii.Number(3),
    })

    // Create an SNS topic for lifecycle hook notifications
    topic := awssns.NewTopic(stack, jsii.String("MyLifecycleHookTopic"), &awssns.TopicProps{})

    // Subscribe an email endpoint to the topic
    topic.AddSubscription(awssns_subscriptions.NewEmailSubscription(jsii.String("example@example.com")))

    // Add a lifecycle hook to the Auto Scaling group
    asg.AddLifecycleHook(jsii.String("MyLifecycleHook"), &awsautoscaling.BasicLifecycleHookProps{
        LifecycleTransition: awsautoscaling.LifecycleTransition_INSTANCE_LAUNCHING,
        NotificationTarget: awsautoscaling.NewSnsTopicHook(topic),
        DefaultResult: awsautoscaling.DefaultResult_CONTINUE,
        HeartbeatTimeout: awscdk.Duration_Minutes(jsii.Number(5)),
    })

    // Output the Auto Scaling group name
    awscdk.NewCfnOutput(stack, jsii.String("AutoScalingGroupName"), &awscdk.CfnOutputProps{
        Value: asg.AutoScalingGroupName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewAutoScalingLifecycleHooksStack(app, "AutoScalingLifecycleHooksStack", &AutoScalingLifecycleHooksStackProps{
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
#### Configuring Notifications for Lifecycle Hooks
You can configure notifications for lifecycle hooks to receive alerts when instances are launched or terminated.

**Example Use Case: Configuring Notifications for Lifecycle Hooks**
The example above demonstrates how to configure an SNS topic to receive notifications for lifecycle hooks.

**Suspending and Resuming Scaling Processes**
You can suspend and resume scaling processes for an Auto Scaling group to temporarily stop scaling activities.

**Example Use Case: Suspending and Resuming Scaling Processes**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsautoscaling"
    "github.com/constructs-go/constructs/v10"
)

type AutoScalingSuspendResumeStackProps struct {
    awscdk.StackProps
}

func NewAutoScalingSuspendResumeStack(scope constructs.Construct, id string, props *AutoScalingSuspendResumeStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an Auto Scaling group
    asg := awsautoscaling.NewAutoScalingGroup(stack, jsii.String("MyAutoScalingGroup"), &awsautoscaling.AutoScalingGroupProps{
        Vpc: vpc,
        InstanceType: awsec2.InstanceType_Of(awsec2.InstanceClass_BURSTABLE2, awsec2.InstanceSize_MICRO),
        MachineImage: awsec2.NewAmazonLinuxImage(&awsec2.AmazonLinuxImageProps{}),
        MinCapacity: jsii.Number(1),
        MaxCapacity: jsii.Number(3),
    })

    // Suspend scaling processes
    asg.ScaleOnSchedule(jsii.String("SuspendScaling"), &awsautoscaling.ScalingSchedule{
        Schedule: awsautoscaling.Schedule_Cron(&awsautoscaling.CronOptions{
            Hour: jsii.String("0"),
            Minute: jsii.String("0"),
        }),
        MinCapacity: jsii.Number(0),
    })

    // Resume scaling processes
    asg.ScaleOnSchedule(jsii.String("ResumeScaling"), &awsautoscaling.ScalingSchedule{
        Schedule: awsautoscaling.Schedule_Cron(&awsautoscaling.CronOptions{
            Hour: jsii.String("6"),
            Minute: jsii.String("0"),
        }),
        MinCapacity: jsii.Number(1),
    })

    // Output the Auto Scaling group name
    awscdk.NewCfnOutput(stack, jsii.String("AutoScalingGroupName"), &awscdk.CfnOutputProps{
        Value: asg.AutoScalingGroupName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewAutoScalingSuspendResumeStack(app, "AutoScalingSuspendResumeStack", &AutoScalingSuspendResumeStackProps{
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

#### Some Limitations to Remember for Amazon EC2 Auto Scaling Group
- Instance Limits: The number of instances in an Auto Scaling group is limited by your EC2 instance limits.
- Scaling Limits: There are limits on the number of scaling policies and scheduled actions you can create.
- Cooldown Period: The cooldown period can affect how quickly your Auto Scaling group responds to changes in demand.

#### Summary
- Amazon EC2 Auto Scaling: Helps maintain application availability and allows you to automatically add or remove EC2 instances.
- Horizontal and Vertical Scaling: Horizontal scaling adds more instances, while vertical scaling increases the size of existing instances.
- Components of an Auto Scaling Group: Launch configuration/template, Auto Scaling group, and scaling policies.
- Types of Scaling Policies: Target tracking, step scaling, and simple scaling.
- Lifecycle Hooks: Enable custom actions during instance launch or termination.
- Configuring Notifications: Use SNS to receive notifications for lifecycle hooks.
- Suspending and Resuming Scaling Processes: Temporarily stop scaling activities.
- Limitations: Instance limits, scaling limits, and cooldown periods.

