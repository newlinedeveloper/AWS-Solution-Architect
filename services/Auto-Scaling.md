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

# AWS Auto Scaling Policies

AWS Auto Scaling allows you to automatically scale your resources (such as EC2 instances, ECS tasks, DynamoDB tables, etc.) to meet demand based on various scaling policies. These policies help to ensure that you have the right amount of capacity to handle the load, while minimizing costs.

## Types of Auto Scaling Policies

1. **Target Tracking Scaling Policy**
2. **Step Scaling Policy**
3. **Scheduled Scaling Policy**

---

### 1. **Target Tracking Scaling Policy**

A target tracking scaling policy works like a thermostat. You set a target value (such as CPU utilization or request count per target) and AWS will automatically adjust the scaling to maintain that target.

- **Example Use Case**: Scale EC2 instances in an Auto Scaling group to maintain an average CPU utilization of 50%.

#### AWS CDK (Golang) Code:

```go
package main

import (
	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awsautoscaling"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type AutoScalingStackProps struct {
	awscdk.StackProps
}

func NewAutoScalingStack(scope constructs.Construct, id string, props *AutoScalingStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}

	stack := awscdk.NewStack(scope, &id, &sprops)

	vpc := awsec2.NewVpc(stack, jsii.String("MyVpc"), nil)

	autoScalingGroup := awsautoscaling.NewAutoScalingGroup(stack, jsii.String("MyASG"), &awsautoscaling.AutoScalingGroupProps{
		Vpc:            vpc,
		InstanceType:   awsec2.InstanceType_Of(awsec2.InstanceClass_BURSTABLE2, awsec2.InstanceSize_MICRO),
		MachineImage:   awsec2.NewAmazonLinuxImage(nil),
		MinCapacity:    jsii.Number(1),
		MaxCapacity:    jsii.Number(10),
	})

	autoScalingGroup.ScaleOnCpuUtilization(jsii.String("KeepCpu50Percent"), &awsautoscaling.CpuUtilizationScalingProps{
		TargetUtilizationPercent: jsii.Number(50),
	})

	return stack
}

func main() {
	app := awscdk.NewApp(nil)
	NewAutoScalingStack(app, "AutoScalingStack", &AutoScalingStackProps{})
	app.Synth(nil)
}
```

---

### 2. **Step Scaling Policy**

In a step scaling policy, you define thresholds for scaling based on CloudWatch alarms. For example, if CPU utilization exceeds 70%, add 2 instances. If it falls below 30%, remove 1 instance.

- **Example Use Case**: Increase the number of instances by 2 if CPU utilization exceeds 70%, decrease by 1 if below 30%.

#### AWS CDK (Golang) Code:

```go
package main

import (
	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awsautoscaling"
	"github.com/aws/aws-cdk-go/awscdk/v2/awscloudwatch"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type StepScalingStackProps struct {
	awscdk.StackProps
}

func NewStepScalingStack(scope constructs.Construct, id string, props *StepScalingStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}

	stack := awscdk.NewStack(scope, &id, &sprops)

	vpc := awsec2.NewVpc(stack, jsii.String("MyVpc"), nil)

	autoScalingGroup := awsautoscaling.NewAutoScalingGroup(stack, jsii.String("MyASG"), &awsautoscaling.AutoScalingGroupProps{
		Vpc:            vpc,
		InstanceType:   awsec2.InstanceType_Of(awsec2.InstanceClass_BURSTABLE2, awsec2.InstanceSize_MICRO),
		MachineImage:   awsec2.NewAmazonLinuxImage(nil),
		MinCapacity:    jsii.Number(1),
		MaxCapacity:    jsii.Number(10),
	})

	cpuAlarmHigh := awscloudwatch.NewAlarm(stack, jsii.String("HighCpuAlarm"), &awscloudwatch.AlarmProps{
		Metric: autoScalingGroup.MetricCpuUtilization(nil),
		Threshold: jsii.Number(70),
		EvaluationPeriods: jsii.Number(1),
	})

	cpuAlarmLow := awscloudwatch.NewAlarm(stack, jsii.String("LowCpuAlarm"), &awscloudwatch.AlarmProps{
		Metric: autoScalingGroup.MetricCpuUtilization(nil),
		Threshold: jsii.Number(30),
		EvaluationPeriods: jsii.Number(1),
		ComparisonOperator: awscloudwatch.ComparisonOperator_LESS_THAN_THRESHOLD,
	})

	autoScalingGroup.ScaleOnMetric(jsii.String("ScaleUpOnHighCpu"), &awsautoscaling.BasicStepScalingPolicyProps{
		Metric: autoScalingGroup.MetricCpuUtilization(nil),
		ScalingSteps: &[]*awsautoscaling.ScalingInterval{
			{
				Upper: jsii.Number(70),
				Change: jsii.Number(2),
			},
		},
	})

	autoScalingGroup.ScaleOnMetric(jsii.String("ScaleDownOnLowCpu"), &awsautoscaling.BasicStepScalingPolicyProps{
		Metric: autoScalingGroup.MetricCpuUtilization(nil),
		ScalingSteps: &[]*awsautoscaling.ScalingInterval{
			{
				Lower: jsii.Number(30),
				Change: jsii.Number(-1),
			},
		},
	})

	return stack
}

func main() {
	app := awscdk.NewApp(nil)
	NewStepScalingStack(app, "StepScalingStack", &StepScalingStackProps{})
	app.Synth(nil)
}
```

---

### 3. **Scheduled Scaling Policy**

Scheduled scaling allows you to scale your resources based on a specific schedule. You can define when to increase or decrease the number of instances based on predictable load patterns (e.g., peak hours or low traffic periods).

- **Example Use Case**: Scale up the number of instances to 5 every day at 8 AM, and scale down to 2 at 6 PM.

#### AWS CDK (Golang) Code:

```go
package main

import (
	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awsautoscaling"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type ScheduledScalingStackProps struct {
	awscdk.StackProps
}

func NewScheduledScalingStack(scope constructs.Construct, id string, props *ScheduledScalingStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}

	stack := awscdk.NewStack(scope, &id, &sprops)

	vpc := awsec2.NewVpc(stack, jsii.String("MyVpc"), nil)

	autoScalingGroup := awsautoscaling.NewAutoScalingGroup(stack, jsii.String("MyASG"), &awsautoscaling.AutoScalingGroupProps{
		Vpc:            vpc,
		InstanceType:   awsec2.InstanceType_Of(awsec2.InstanceClass_BURSTABLE2, awsec2.InstanceSize_MICRO),
		MachineImage:   awsec2.NewAmazonLinuxImage(nil),
		MinCapacity:    jsii.Number(1),
		MaxCapacity:    jsii.Number(10),
	})

	autoScalingGroup.ScaleOnSchedule(jsii.String("ScaleUpInTheMorning"), &awsautoscaling.ScalingSchedule{
		Schedule: awsautoscaling.Schedule_Cron(&awsautoscaling.CronOptions{
			Hour: jsii.String("8"),
			Minute: jsii.String("0"),
		}),
		DesiredCapacity: jsii.Number(5),
	})

	autoScalingGroup.ScaleOnSchedule(jsii.String("ScaleDownInTheEvening"), &awsautoscaling.ScalingSchedule{
		Schedule: awsautoscaling.Schedule_Cron(&awsautoscaling.CronOptions{
			Hour: jsii.String("18"),
			Minute: jsii.String("0"),
		}),
		DesiredCapacity: jsii.Number(2),
	})

	return stack
}

func main() {
	app := awscdk.NewApp(nil)
	NewScheduledScalingStack(app, "ScheduledScalingStack", &ScheduledScalingStackProps{})
	app.Synth(nil)
}
```

---

## Conclusion

These are the three main types of AWS Auto Scaling policies:
- **Target Tracking Scaling** for maintaining specific metric values like CPU utilization.
- **Step Scaling** for triggering based on CloudWatch alarms when crossing defined thresholds.
- **Scheduled Scaling** for scaling based on predictable, scheduled events.


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

