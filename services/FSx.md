# Amazon FSx

Amazon FSx provides fully managed third-party file systems that are optimized for a variety of workloads. It offers two main types of file systems: Amazon FSx for Lustre and Amazon FSx for Windows File Server.

## Amazon FSx for Lustre vs Amazon FSx for Windows File Server

### Amazon FSx for Lustre

- **Purpose**: Optimized for high-performance computing (HPC) and machine learning workloads that require fast storage.
- **Performance**: Provides sub-millisecond latencies and high throughput.
- **Use Case**: Ideal for workloads that require fast processing of large datasets, such as big data analytics, media processing, and genomic analysis.

### Amazon FSx for Windows File Server

- **Purpose**: Provides fully managed Windows file systems built on Windows Server.
- **Performance**: Optimized for enterprise applications that require shared file storage with Windows compatibility.
- **Use Case**: Ideal for Windows-based applications that require file storage, such as content management, home directories, and web serving.

## Example Use Case: Creating an Amazon FSx for Lustre File System

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsfsx"
    "github.com/constructs-go/constructs/v10"
)

type FSxLustreStackProps struct {
    awscdk.StackProps
}

func NewFSxLustreStack(scope constructs.Construct, id string, props *FSxLustreStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an Amazon FSx for Lustre file system
    fsxLustre := awsfsx.NewLustreFileSystem(stack, jsii.String("MyFSxLustre"), &awsfsx.LustreFileSystemProps{
        Vpc: vpc,
        StorageCapacityGiB: jsii.Number(1200),
        DeploymentType: awsfsx.LustreDeploymentType_SCRATCH_2,
    })

    // Output the file system ID
    awscdk.NewCfnOutput(stack, jsii.String("FileSystemID"), &awscdk.CfnOutputProps{
        Value: fsxLustre.FileSystemId(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewFSxLustreStack(app, "FSxLustreStack", &FSxLustreStackProps{
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

**Example Use Case: Creating an Amazon FSx for Windows File Server**
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsfsx"
    "github.com/constructs-go/constructs/v10"
)

type FSxWindowsStackProps struct {
    awscdk.StackProps
}

func NewFSxWindowsStack(scope constructs.Construct, id string, props *FSxWindowsStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an Amazon FSx for Windows File Server
    fsxWindows := awsfsx.NewWindowsFileSystem(stack, jsii.String("MyFSxWindows"), &awsfsx.WindowsFileSystemProps{
        Vpc: vpc,
        StorageCapacityGiB: jsii.Number(1200),
        ThroughputCapacityMibps: jsii.Number(32),
        WindowsConfiguration: &awsfsx.WindowsFileSystemConfiguration{
            ActiveDirectoryId: jsii.String("d-1234567890"),
            ThroughputCapacity: jsii.Number(32),
        },
    })

    // Output the file system ID
    awscdk.NewCfnOutput(stack, jsii.String("FileSystemID"), &awscdk.CfnOutputProps{
        Value: fsxWindows.FileSystemId(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewFSxWindowsStack(app, "FSxWindowsStack", &FSxWindowsStackProps{
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

- Amazon FSx: Provides fully managed third-party file systems optimized for various workloads.
- Amazon FSx for Lustre: Optimized for high-performance computing and machine learning workloads that require fast storage.
- Amazon FSx for Windows File Server: Provides fully managed Windows file systems built on Windows Server, optimized for enterprise applications that require shared file storage with Windows compatibility.
- Example Use Cases: Creating an Amazon FSx for Lustre file system and an Amazon FSx for Windows File Server using AWS CDK in Go.
