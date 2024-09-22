# AWS Backup

AWS Backup is a fully managed backup service that makes it easy to centralize and automate the backup of data across AWS services. It provides a unified backup solution to protect your AWS resources, such as Amazon EC2 instances, Amazon RDS databases, Amazon EFS file systems, and more.

## Backup Retention Period Too Short?

If your backup retention period is too short, you risk losing critical data if a backup is deleted before you need it. AWS Backup allows you to configure backup plans with custom retention periods to ensure that your data is retained for the required duration.

### Example Use Case: Creating a Backup Plan with a Longer Retention Period

In this example, we will create a backup plan using AWS CDK in Go with a retention period of 30 days.

### AWS CDK Code for Creating a Backup Plan

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsbackup"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsevents"
    "github.com/constructs-go/constructs/v10"
)

type BackupPlanStackProps struct {
    awscdk.StackProps
}

func NewBackupPlanStack(scope constructs.Construct, id string, props *BackupPlanStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a backup vault
    vault := awsbackup.NewBackupVault(stack, jsii.String("MyBackupVault"), &awsbackup.BackupVaultProps{
        BackupVaultName: jsii.String("MyBackupVault"),
    })

    // Create a backup plan
    plan := awsbackup.NewBackupPlan(stack, jsii.String("MyBackupPlan"), &awsbackup.BackupPlanProps{
        BackupPlanName: jsii.String("MyBackupPlan"),
    })

    // Add a rule to the backup plan with a retention period of 30 days
    plan.AddRule(&awsbackup.BackupPlanRule{
        RuleName:         jsii.String("DailyBackup"),
        ScheduleExpression: awsevents.Schedule_Rate(awscdk.Duration_Days(jsii.Number(1))),
        BackupVault:      vault,
        DeleteAfter:      awscdk.Duration_Days(jsii.Number(30)),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewBackupPlanStack(app, "BackupPlanStack", &BackupPlanStackProps{
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
- AWS Backup: A fully managed backup service that centralizes and automates the backup of data across AWS services.
- Backup Retention Period: Ensures that your data is retained for the required duration to prevent data loss.
- Example Use Case: Creating a backup plan with a retention period of 30 days using AWS CDK in Go.
