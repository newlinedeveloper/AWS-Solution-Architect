# Amazon EC2

Amazon Elastic Compute Cloud (EC2) provides scalable computing capacity in the AWS cloud. Using Amazon EC2 eliminates the need to invest in hardware upfront, so you can develop and deploy applications faster.

## Components of an EC2 Instance

- **AMI (Amazon Machine Image)**: A template that contains the software configuration (operating system, application server, and applications) required to launch your instance.
- **Instance Type**: Specifies the hardware of the host computer used for your instance.
- **Instance Store Volumes**: Temporary block-level storage for your instance.
- **Elastic Block Store (EBS) Volumes**: Persistent block-level storage for your instance.
- **Key Pairs**: Secure login information for your instance.
- **Security Groups**: Act as a virtual firewall to control inbound and outbound traffic.

## Types of EC2 Instances

- **General Purpose**: Balanced compute, memory, and networking resources (e.g., t3, m5).
- **Compute Optimized**: Ideal for compute-bound applications (e.g., c5).
- **Memory Optimized**: Designed for memory-intensive applications (e.g., r5).
- **Storage Optimized**: High, sequential read and write access to large datasets (e.g., i3).
- **Accelerated Computing**: Use hardware accelerators, or co-processors, to perform functions such as floating-point number calculations, graphics processing, or data pattern matching (e.g., p3).

## Storage with Highest IOPS for EC2 Instance

- **Instance Store Volumes**: Provide high, sequential read and write access to large datasets.
- **Provisioned IOPS SSD (io1/io2)**: Designed for I/O-intensive workloads that require low latency and high throughput.

## Instance Purchasing Options

- **On-Demand Instances**: Pay for compute capacity by the hour or second with no long-term commitments.
- **Reserved Instances**: Provide a significant discount (up to 75%) compared to On-Demand pricing.
- **Spot Instances**: Allow you to request spare Amazon EC2 computing capacity for up to 90% off the On-Demand price.
- **Dedicated Hosts**: Physical servers with EC2 instance capacity fully dedicated to your use.

## Comparison of Different Types of EC2 Health Checks

- **EC2 Status Checks**: Monitor the health of the underlying hardware and software.
- **ELB Health Checks**: Monitor the health of the instances behind a load balancer.
- **Auto Scaling Health Checks**: Monitor the health of instances in an Auto Scaling group.

## EC2 Placement Groups

- **Cluster Placement Group**: Packs instances close together inside an Availability Zone.
- **Spread Placement Group**: Strictly places a small group of instances across distinct underlying hardware.
- **Partition Placement Group**: Divides each group into logical segments called partitions.

## Security Groups and Network Access Control Lists (ACLs)

- **Security Groups**: Act as a virtual firewall for your instance to control inbound and outbound traffic.
- **Network ACLs**: Act as a firewall for controlling traffic in and out of one or more subnets.

## Example Use Case: Creating an EC2 Instance with a Security Group

### AWS CDK Code for Creating an EC2 Instance with a Security Group

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type EC2InstanceStackProps struct {
    awscdk.StackProps
}

func NewEC2InstanceStack(scope constructs.Construct, id string, props *EC2InstanceStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create a security group
    securityGroup := awsec2.NewSecurityGroup(stack, jsii.String("MySecurityGroup"), &awsec2.SecurityGroupProps{
        Vpc: vpc,
        Description: jsii.String("Allow SSH and HTTP traffic"),
        AllowAllOutbound: jsii.Bool(true),
    })

    // Allow SSH access from anywhere
    securityGroup.AddIngressRule(awsec2.Peer_AnyIpv4(), awsec2.Port_Tcp(jsii.Number(22)), jsii.String("Allow SSH access from anywhere"))

    // Allow HTTP access from anywhere
    securityGroup.AddIngressRule(awsec2.Peer_AnyIpv4(), awsec2.Port_Tcp(jsii.Number(80)), jsii.String("Allow HTTP access from anywhere"))

    // Create an EC2 instance
    instance := awsec2.NewInstance(stack, jsii.String("MyInstance"), &awsec2.InstanceProps{
        Vpc: vpc,
        InstanceType: awsec2.InstanceType_Of(awsec2.InstanceClass_BURSTABLE2, awsec2.InstanceSize_MICRO),
        MachineImage: awsec2.NewAmazonLinuxImage(&awsec2.AmazonLinuxImageProps{}),
        SecurityGroup: securityGroup,
        KeyName: jsii.String("my-key-pair"),
    })

    // Output the instance ID
    awscdk.NewCfnOutput(stack, jsii.String("InstanceId"), &awscdk.CfnOutputProps{
        Value: instance.InstanceId(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewEC2InstanceStack(app, "EC2InstanceStack", &EC2InstanceStackProps{
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

- Amazon EC2: Provides scalable computing capacity in the AWS cloud.
- Components of an EC2 Instance: AMI, instance type, instance store volumes, EBS volumes, key pairs, and security groups.
- Types of EC2 Instances: General purpose, compute optimized, memory optimized, storage optimized, and accelerated computing.
- Storage with Highest IOPS: Instance store volumes and Provisioned IOPS SSD (io1/io2).
- Instance Purchasing Options: On-Demand, Reserved, Spot, and Dedicated Hosts.
- EC2 Health Checks: EC2 status checks, ELB health checks, and Auto Scaling health checks.
- EC2 Placement Groups: Cluster, spread, and partition placement groups.
- Security Groups and Network ACLs: Security groups act as a virtual firewall for instances, while network ACLs control traffic in and out of subnets.
