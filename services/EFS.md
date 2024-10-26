# Amazon Elastic File System (EFS)

Amazon Elastic File System (EFS) is a scalable, fully managed elastic NFS file system for use with AWS Cloud services and on-premises resources. It is designed to provide scalable file storage for use with Amazon EC2 instances.

## How To Mount An Amazon EFS File System

Mounting an EFS file system to an EC2 instance allows you to access the file system from your instance. You can mount the file system using the EFS mount helper or the standard NFS client.

### Example Use Case: Creating and Mounting an EFS File System

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsefs"
    "github.com/constructs-go/constructs/v10"
)

type EFSStackProps struct {
    awscdk.StackProps
}

func NewEFSStack(scope constructs.Construct, id string, props *EFSStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an EFS file system
    fileSystem := awsefs.NewFileSystem(stack, jsii.String("MyEFS"), &awsefs.FileSystemProps{
        Vpc: vpc,
        LifecyclePolicy: awsefs.LifecyclePolicy_AFTER_7_DAYS,
        PerformanceMode: awsefs.PerformanceMode_GENERAL_PURPOSE,
    })

    // Create a security group for the EC2 instance
    securityGroup := awsec2.NewSecurityGroup(stack, jsii.String("EFSSecurityGroup"), &awsec2.SecurityGroupProps{
        Vpc: vpc,
    })

    // Create an EC2 instance
    instance := awsec2.NewInstance(stack, jsii.String("MyInstance"), &awsec2.InstanceProps{
        InstanceType: awsec2.InstanceType_Of(awsec2.InstanceClass_BURSTABLE2, awsec2.InstanceSize_MICRO),
        MachineImage: awsec2.MachineImage_LatestAmazonLinux(),
        Vpc: vpc,
        SecurityGroup: securityGroup,
    })

    // Output the file system ID and instance ID
    awscdk.NewCfnOutput(stack, jsii.String("FileSystemID"), &awscdk.CfnOutputProps{
        Value: fileSystem.FileSystemId(),
    })
    awscdk.NewCfnOutput(stack, jsii.String("InstanceID"), &awscdk.CfnOutputProps{
        Value: instance.InstanceId(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewEFSStack(app, "EFSStack", &EFSStackProps{
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

#### Mounting the EFS File System

**Connect to EC2 instance**
```
ssh -i "your-key-pair.pem" ec2-user@your-instance-public-dns
```
**Install the EFS mount helper:**
```
sudo yum install -y amazon-efs-utils
```

**Create a directory and mount the file system:**
```
sudo mkdir /mnt/efs
sudo mount -t efs fs-12345678:/ /mnt/efs
```
#### EFS-to-EFS Regional Data Transfer

Amazon EFS supports data transfer between file systems in different AWS regions. This can be useful for disaster recovery, data migration, or data replication.

Example Use Case: EFS-to-EFS Data Transfer
To transfer data between two EFS file systems in different regions, you can use AWS DataSync.

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsefs"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsdatasync"
    "github.com/constructs-go/constructs/v10"
)

type EFSDataTransferStackProps struct {
    awscdk.StackProps
}

func NewEFSDataTransferStack(scope constructs.Construct, id string, props *EFSDataTransferStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create source and destination EFS file systems
    sourceFileSystem := awsefs.NewFileSystem(stack, jsii.String("SourceEFS"), &awsefs.FileSystemProps{
        Vpc: vpc,
    })
    destinationFileSystem := awsefs.NewFileSystem(stack, jsii.String("DestinationEFS"), &awsefs.FileSystemProps{
        Vpc: vpc,
    })

    // Create DataSync task
    sourceLocation := awsdatasync.NewCfnLocationEFS(stack, jsii.String("SourceLocation"), &awsdatasync.CfnLocationEFSProps{
        Ec2Config: &awsdatasync.CfnLocationEFS_Ec2ConfigProperty{
            SecurityGroupArns: &[]*string{sourceFileSystem.FileSystemId()},
            SubnetArn: vpc.PrivateSubnets()[0].SubnetArn(),
        },
        EfsFilesystemArn: sourceFileSystem.FileSystemArn(),
    })
    destinationLocation := awsdatasync.NewCfnLocationEFS(stack, jsii.String("DestinationLocation"), &awsdatasync.CfnLocationEFSProps{
        Ec2Config: &awsdatasync.CfnLocationEFS_Ec2ConfigProperty{
            SecurityGroupArns: &[]*string{destinationFileSystem.FileSystemId()},
            SubnetArn: vpc.PrivateSubnets()[0].SubnetArn(),
        },
        EfsFilesystemArn: destinationFileSystem.FileSystemArn(),
    })
    task := awsdatasync.NewCfnTask(stack, jsii.String("DataSyncTask"), &awsdatasync.CfnTaskProps{
        SourceLocationArn: sourceLocation.AttrLocationArn(),
        DestinationLocationArn: destinationLocation.AttrLocationArn(),
    })

    // Output the task ID
    awscdk.NewCfnOutput(stack, jsii.String("TaskID"), &awscdk.CfnOutputProps{
        Value: task.Ref(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewEFSDataTransferStack(app, "EFSDataTransferStack", &EFSDataTransferStackProps{
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
#### Amazon EFS Storage Lifecycle
Amazon EFS provides a storage lifecycle management feature that automatically moves files between the Standard and Infrequent Access (IA) storage classes based on the last time they were accessed. This helps reduce storage costs for infrequently accessed files.

**Example Use Case: Configuring EFS Storage Lifecycle**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsefs"
    "github.com/constructs-go/constructs/v10"
)

type EFSStorageLifecycleStackProps struct {
    awscdk.StackProps
}

func NewEFSStorageLifecycleStack(scope constructs.Construct, id string, props *EFSStorageLifecycleStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an EFS file system with lifecycle policy
    fileSystem := awsefs.NewFileSystem(stack, jsii.String("MyEFS"), &awsefs.FileSystemProps{
        Vpc: vpc,
        LifecyclePolicy: awsefs.LifecyclePolicy_AFTER_30_DAYS,
        PerformanceMode: awsefs.PerformanceMode_GENERAL_PURPOSE,
    })

    // Output the file system ID
    awscdk.NewCfnOutput(stack, jsii.String("FileSystemID"), &awscdk.CfnOutputProps{
        Value: fileSystem.FileSystemId(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewEFSStorageLifecycleStack(app, "EFSStorageLifecycleStack", &EFSStorageLifecycleStackProps{
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

- Amazon EFS: A scalable, fully managed elastic NFS file system for use with AWS Cloud services and on-premises resources.
- Mounting an EFS File System: Allows you to access the file system from your EC2 instances.
- EFS-to-EFS Regional Data Transfer: Supports data transfer between file systems in different AWS regions using AWS DataSync.

To improve the performance of big data processing workflows running on **Amazon Elastic File System (Amazon EFS)**, the correct solution depends on the specific performance needs of the workload. 

For big data processing, which typically requires high throughput and the ability to handle large volumes of data efficiently, **Amazon EFS Throughput Mode** and **Performance Mode** are crucial considerations.

### The two relevant options are:

1. **EFS Max I/O Performance Mode**:
   - **When to use**: 
     - For highly parallelized applications, such as big data analysis, media processing, or genomics workflows, which require the highest levels of scalability.
     - It is designed for workloads where thousands of clients need access to the file system simultaneously.
     - **Tradeoff**: Slightly higher latencies but offers the ability to handle large amounts of data and clients in parallel.
   - **How it helps**: Allows you to scale the file system with very high levels of throughput and connections from multiple EC2 instances without performance degradation.
  
2. **Provisioned Throughput Mode**:
   - **When to use**:
     - If the workload has specific and predictable throughput requirements that exceed what is provided by default.
     - In scenarios where consistent and higher levels of throughput are required for processing large datasets.
     - **How it helps**: You can specify the throughput you need, independent of the amount of data stored, ensuring that big data workflows get sufficient throughput for processing tasks.

---

### **Correct Solution**:
For an **analytics company running big data processing workflows**, the most suitable choice is likely the **EFS Max I/O Performance Mode**, since it provides high scalability and can handle the large, parallelized nature of big data workflows. If predictable throughput is also important, combining **Max I/O mode** with **Provisioned Throughput Mode** may be necessary for consistent performance.


