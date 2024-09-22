# Amazon Elastic Block Store (EBS)

Amazon Elastic Block Store (EBS) provides persistent block storage volumes for use with Amazon EC2 instances. Each Amazon EBS volume is automatically replicated within its Availability Zone to protect you from component failure, offering high availability and durability.

## SSD vs HDD Type Volumes

### SSD (Solid State Drive) Volumes

- **General Purpose SSD (gp2/gp3)**: Balanced price and performance for a wide variety of workloads.
- **Provisioned IOPS SSD (io1/io2)**: Highest performance SSD volume designed for latency-sensitive transactional workloads.

### HDD (Hard Disk Drive) Volumes

- **Throughput Optimized HDD (st1)**: Low-cost HDD volume designed for frequently accessed, throughput-intensive workloads.
- **Cold HDD (sc1)**: Lowest cost HDD volume designed for less frequently accessed workloads.

## Amazon EBS Multi-Attach Feature

The EBS Multi-Attach feature allows a single Provisioned IOPS SSD (io1/io2) volume to be concurrently attached to up to 16 Nitro-based EC2 instances within the same Availability Zone. This feature is ideal for applications that require high availability and need to access the same data from multiple instances.

### Example Use Case: Creating an EBS Volume with Multi-Attach

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type EBSMultiAttachStackProps struct {
    awscdk.StackProps
}

func NewEBSMultiAttachStack(scope constructs.Construct, id string, props *EBSMultiAttachStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an EBS volume with Multi-Attach enabled
    volume := awsec2.NewVolume(stack, jsii.String("MyEBSVolume"), &awsec2.VolumeProps{
        AvailabilityZone: jsii.String("us-west-2a"),
        Size: awscdk.Size_Gibibytes(jsii.Number(100)),
        VolumeType: awsec2.EbsDeviceVolumeType_IO1,
        Iops: jsii.Number(10000),
        MultiAttachEnabled: jsii.Bool(true),
    })

    // Output the volume ID
    awscdk.NewCfnOutput(stack, jsii.String("VolumeID"), &awscdk.CfnOutputProps{
        Value: volume.VolumeId(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewEBSMultiAttachStack(app, "EBSMultiAttachStack", &EBSMultiAttachStackProps{
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
#### Amazon EBS Copy Snapshots
Amazon EBS allows you to create point-in-time snapshots of your volumes, which are stored in Amazon S3. You can copy these snapshots across AWS regions and accounts for backup, disaster recovery, and migration purposes.

**Example Use Case: Copying an EBS Snapshot**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type EBSCopySnapshotStackProps struct {
    awscdk.StackProps
}

func NewEBSCopySnapshotStack(scope constructs.Construct, id string, props *EBSCopySnapshotStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an EBS volume
    volume := awsec2.NewVolume(stack, jsii.String("MyEBSVolume"), &awsec2.VolumeProps{
        AvailabilityZone: jsii.String("us-west-2a"),
        Size: awscdk.Size_Gibibytes(jsii.Number(100)),
        VolumeType: awsec2.EbsDeviceVolumeType_GP2,
    })

    // Create a snapshot of the volume
    snapshot := volume.Snapshot(jsii.String("MySnapshot"))

    // Copy the snapshot to another region
    copiedSnapshot := awsec2.NewCfnSnapshot(stack, jsii.String("CopiedSnapshot"), &awsec2.CfnSnapshotProps{
        SourceRegion: jsii.String("us-west-2"),
        SourceSnapshotId: snapshot.SnapshotId(),
    })

    // Output the copied snapshot ID
    awscdk.NewCfnOutput(stack, jsii.String("CopiedSnapshotID"), &awscdk.CfnOutputProps{
        Value: copiedSnapshot.Ref(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewEBSCopySnapshotStack(app, "EBSCopySnapshotStack", &EBSCopySnapshotStackProps{
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

- Amazon EBS: Provides persistent block storage volumes for use with Amazon EC2 instances.
- SSD vs HDD Type Volumes: SSD volumes (gp2/gp3, io1/io2) are optimized for performance, while HDD volumes (st1, sc1) are optimized for throughput and cost.
- EBS Multi-Attach Feature: Allows a single Provisioned IOPS SSD (io1/io2) volume to be concurrently attached to multiple Nitro-based EC2 instances within the same Availability Zone.
- Copy Snapshots: Allows you to create point-in-time snapshots of your volumes and copy them across AWS regions and accounts for backup, disaster recovery, and migration purposes.
