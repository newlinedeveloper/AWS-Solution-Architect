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

AWS EC2 (Elastic Compute Cloud) offers a broad range of instance types, purchase options, and placement group strategies to meet various workloads. Below is an in-depth explanation of each aspect, including the EC2 instance types, purchase options, and placement groups, as well as their use cases.

---

## **1. EC2 Instance Types**

EC2 instances are grouped into different **instance families**, each optimized for different types of applications. Each family offers different combinations of CPU, memory, storage, and networking capabilities to match your workloads. Here's an overview of the main types:

### **a. General Purpose Instances**
These instances are balanced across compute, memory, and networking, making them suitable for a variety of applications.

| Instance Family | Description | Use Cases | Examples |
|-----------------|-------------|-----------|----------|
| **T-Series** (T4g, T3, T3a) | Burstable instances for variable workloads | Web servers, small databases, development environments | T3.micro, T4g.large |
| **M-Series** (M7g, M6i, M6g, M5, M4) | Balanced compute, memory, and networking | Application servers, web servers, enterprise applications | M6i.large, M5.xlarge |

### **b. Compute Optimized Instances**
Designed for compute-bound applications that require high-performance processors.

| Instance Family | Description | Use Cases | Examples |
|-----------------|-------------|-----------|----------|
| **C-Series** (C7g, C6i, C6g, C5, C4) | Optimized for compute-intensive workloads | High-performance computing, gaming servers, batch processing | C6i.xlarge, C5.4xlarge |

### **c. Memory Optimized Instances**
These instances are ideal for memory-bound applications that require fast access to large datasets.

| Instance Family | Description | Use Cases | Examples |
|-----------------|-------------|-----------|----------|
| **R-Series** (R7g, R6i, R6g, R5, R4) | High memory for memory-intensive applications | In-memory databases (e.g., Redis, SAP), big data analytics | R6g.large, R5.2xlarge |
| **X-Series** (X2gd, X1, X1e) | Extreme memory-optimized | In-memory databases, real-time big data processing | X1.32xlarge, X2gd.xlarge |
| **High Memory** | Up to 12 TB of memory | SAP HANA, in-memory databases | u-24tb1.metal |

### **d. Storage Optimized Instances**
Optimized for workloads requiring high sequential read/write access to large datasets on local storage.

| Instance Family | Description | Use Cases | Examples |
|-----------------|-------------|-----------|----------|
| **I-Series** (I4i, I3) | High-performance NVMe storage | NoSQL databases (e.g., Cassandra), data warehousing | I3.large, I4i.xlarge |
| **D-Series** (D3, D2) | Dense HDD storage | Data-intensive applications, data lakes, Hadoop clusters | D3.xlarge, D2.8xlarge |

### **e. GPU Instances**
Optimized for graphics processing (GPU) and machine learning (ML) workloads.

| Instance Family | Description | Use Cases | Examples |
|-----------------|-------------|-----------|----------|
| **P-Series** (P4, P3, P2) | High-performance GPUs for ML, AI, and scientific computation | ML model training, graphics rendering, high-performance computing | P3.2xlarge, P4d.24xlarge |
| **G-Series** (G5, G4ad, G4dn) | GPU-based instances for graphics-heavy applications | Video rendering, game streaming, AI inference | G4dn.xlarge, G5.12xlarge |

### **f. High-Performance Computing (HPC) Instances**
Designed for scientific modeling, simulations, and high-performance computing workloads.

| Instance Family | Description | Use Cases | Examples |
|-----------------|-------------|-----------|----------|
| **HPC** (Hpc6a) | HPC-optimized instances | Scientific simulations, seismic analysis, financial modeling | Hpc6a.48xlarge |

---

## **2. EC2 Purchase Options**

AWS EC2 provides several purchasing models depending on your workload, cost optimization needs, and flexibility.

### **a. On-Demand Instances**
- **Description**: Pay for compute capacity by the hour or second with no long-term commitments.
- **Use Case**: Ideal for short-term, unpredictable workloads that cannot be interrupted or require flexibility.
- **Pricing**: Higher than other models but with complete flexibility.

### **b. Reserved Instances (RI)**
- **Description**: Provides a significant discount (up to 75%) over On-Demand pricing in exchange for committing to a 1 or 3-year term.
- **Use Case**: Ideal for stable workloads with predictable resource needs.
- **Pricing**: Choose between Standard RIs (largest discount) and Convertible RIs (flexibility to change instance types).

### **c. Spot Instances**
- **Description**: Purchase unused EC2 capacity at up to 90% off On-Demand prices.
- **Use Case**: Best for workloads that are fault-tolerant, stateless, or can be interrupted (e.g., batch processing, big data analytics).
- **Pricing**: Highly variable and based on supply/demand of EC2 capacity.

### **d. Savings Plans**
- **Description**: Flexible pricing model offering lower prices on EC2 in exchange for a commitment to a consistent amount of usage (measured in USD/hour) for 1 or 3 years.
- **Use Case**: Useful for organizations with predictable usage who want flexibility across instance families, sizes, regions, and compute types.
- **Pricing**: Discounts up to 72% off On-Demand pricing.

### **e. Dedicated Hosts**
- **Description**: Physical servers dedicated to your use.
- **Use Case**: Necessary for meeting regulatory or compliance requirements, software licensing that is tied to physical servers.
- **Pricing**: Higher cost due to dedicated resources.

### **f. Dedicated Instances**
- **Description**: Instances running on hardware dedicated to a single customer.
- **Use Case**: Use when you require instances isolated at the hardware level from other customers for compliance or regulatory reasons.
- **Pricing**: More expensive than shared hardware instances.

### **g. Capacity Reservations**
- **Description**: Guarantees capacity in a specific Availability Zone for as long as needed.
- **Use Case**: Ideal for ensuring capacity for critical workloads, especially in environments with unpredictable demand.
- **Pricing**: Charged like On-Demand instances but with guaranteed capacity.

---

## **3. EC2 Placement Groups**

Placement groups in EC2 help control the placement of instances to meet specific performance or fault-tolerance requirements.

### **a. Cluster Placement Group**
- **Description**: Groups instances within a single Availability Zone for low-latency, high-throughput workloads.
- **Use Case**: Ideal for tightly-coupled, high-performance computing (HPC), and networking-intensive applications.
- **Trade-off**: High performance but lower fault tolerance (all instances in a single AZ).

### **b. Spread Placement Group**
- **Description**: Distributes instances across underlying hardware to reduce the likelihood of correlated hardware failures.
- **Use Case**: Use for applications that require high availability and need instances placed across distinct hardware to minimize risk.
- **Trade-off**: Limited to 7 instances per AZ but offers fault isolation.

### **c. Partition Placement Group**
- **Description**: Instances are divided into logical partitions, each partition being isolated from failures in other partitions.
- **Use Case**: Best for large-scale distributed and replicated workloads, such as Hadoop, HDFS, and Cassandra.
- **Trade-off**: Offers greater fault isolation compared to Cluster Placement Groups while allowing scaling.

---

## **Key Considerations for Choosing EC2 Types, Purchase Options, and Placement Groups:**

1. **Workload Characteristics**: The choice of EC2 instance type depends on whether your workload is compute-intensive, memory-intensive, or requires high throughput storage or networking.
   
2. **Cost Optimization**: On-demand is flexible but expensive; reserved instances and savings plans provide long-term savings, while spot instances offer the lowest cost but with potential interruptions.

3. **Availability and Fault Tolerance**: Use Placement Groups to control instance placement based on performance and availability requirements (e.g., Cluster for performance, Spread for fault tolerance).

4. **Networking Needs**: High-performance computing or networking-intensive workloads may benefit from using enhanced networking and placement groups optimized for low-latency communication.

---

### **Summary**:

- **EC2 Instance Types**: Choose from General Purpose, Compute Optimized, Memory Optimized, Storage Optimized, GPU Instances, and HPC Instances depending on your workload.
- **EC2 Purchase Options**: Select On-Demand, Reserved, Spot, Dedicated Hosts/Instances, or Capacity Reservations based on your budget and workload predictability.
- **EC2 Placement Groups**: Use Cluster, Spread, or Partition placement groups depending on whether you need performance optimization, fault tolerance, or isolated workloads.


