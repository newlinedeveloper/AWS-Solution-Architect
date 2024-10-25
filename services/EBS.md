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


Amazon Elastic Block Store (EBS) offers several types of volumes, each designed for different use cases and performance characteristics. Here's a breakdown of the **types of EBS volumes**, their **capacity limits**, and the **ideal use cases** for each type:

### 1. **General Purpose SSD (gp3)**
- **Description**: The latest generation of general-purpose SSD that provides a balance of price and performance. It decouples IOPS and throughput from capacity, allowing for customization.
- **Capacity**: 1 GiB to 16 TiB.
- **Baseline Performance**:
  - Baseline: **3,000 IOPS** and **125 MiB/s**.
  - Maximum Performance: Up to **16,000 IOPS** and **1,000 MiB/s**.
- **Use Case**:
  - General-purpose workloads.
  - Low-latency, random I/O operations (e.g., small databases, virtual desktops, boot volumes).
  - Applications requiring predictable performance with flexible configuration for cost optimization.

### 2. **General Purpose SSD (gp2)**
- **Description**: The previous generation of general-purpose SSD, providing a balance of price and performance with burst capability.
- **Capacity**: 1 GiB to 16 TiB.
- **Baseline Performance**:
  - 3 IOPS per GiB of storage.
  - **Minimum**: 100 IOPS (for small volumes).
  - **Maximum**: Up to **16,000 IOPS** (based on volume size).
- **Burst Performance**:
  - Volumes smaller than 1 TiB can burst up to **3,000 IOPS**.
- **Use Case**:
  - General-purpose workloads with bursty performance requirements (e.g., development environments, small databases, web servers).
  - Use cases where you can benefit from burst performance at a lower cost.

### 3. **Provisioned IOPS SSD (io2 and io2 Block Express)**
- **Description**: High-performance SSD designed for mission-critical, I/O-intensive applications. **io2 Block Express** is the newer, faster version.
- **Capacity**: 
  - **io2**: 4 GiB to 16 TiB.
  - **io2 Block Express**: 4 GiB to 64 TiB.
- **Performance**:
  - **io2**: Up to **64,000 IOPS** and **1,000 MiB/s**.
  - **io2 Block Express**: Up to **256,000 IOPS** and **4,000 MiB/s**.
  - Supports **multi-attach** (multiple instances can connect to the same volume simultaneously).
  - Offers **99.999% durability**.
- **Use Case**:
  - Applications with **high IOPS** and **low-latency** requirements, such as large relational or NoSQL databases (e.g., MySQL, PostgreSQL, Cassandra).
  - Applications requiring high availability, durability, and consistent performance (e.g., SAP HANA, Oracle databases).
  - High-transaction workloads (e.g., enterprise applications, financial systems).

### 4. **Provisioned IOPS SSD (io1)**
- **Description**: Previous generation of high-performance SSD for I/O-intensive workloads.
- **Capacity**: 4 GiB to 16 TiB.
- **Performance**:
  - Up to **64,000 IOPS** and **1,000 MiB/s**.
  - More expensive compared to io2 but with similar use cases.
- **Use Case**: Same as io2 but older and slightly less efficient. io2 is recommended for new workloads due to improved durability and cost-efficiency.

### 5. **Throughput Optimized HDD (st1)**
- **Description**: Low-cost HDD designed for throughput-intensive workloads. It is optimized for large, sequential workloads rather than random access.
- **Capacity**: 125 GiB to 16 TiB.
- **Performance**:
  - Baseline: **40 MiB/s per TiB**.
  - Maximum: Up to **500 MiB/s**.
  - Performance is determined by throughput rather than IOPS.
- **Use Case**:
  - Throughput-intensive workloads with large, sequential read/write operations (e.g., big data analytics, data warehouses, log processing, ETL jobs).
  - Suitable for workloads that need to process large amounts of data sequentially but don’t require high IOPS.

### 6. **Cold HDD (sc1)**
- **Description**: Lowest-cost HDD designed for infrequent, cold data access. It provides the lowest performance among all EBS types.
- **Capacity**: 125 GiB to 16 TiB.
- **Performance**:
  - Baseline: **12 MiB/s per TiB**.
  - Maximum: Up to **250 MiB/s**.
  - Performance is optimized for sequential throughput, not IOPS.
- **Use Case**:
  - Cold data storage where access is infrequent (e.g., backups, archives, large datasets that are rarely accessed).
  - Scenarios where cost is the primary concern over performance.

### 7. **Magnetic (Standard) – Deprecated**
- **Description**: Previously a magnetic storage option, but AWS no longer recommends this type of storage, as it has been effectively replaced by more efficient options like **sc1** and **st1**.
- **Capacity**: 1 GiB to 1 TiB.
- **Performance**:
  - Lower IOPS and throughput compared to modern SSD or HDD options.
- **Use Case**: AWS suggests using other volume types instead.

---

### Summary: Which EBS Volume Type to Use?

| Volume Type                | Capacity Range        | Max IOPS (IOPS/GB) | Max Throughput   | Use Case                                                 |
|----------------------------|-----------------------|--------------------|------------------|----------------------------------------------------------|
| **gp3 (General Purpose SSD)**  | 1 GiB to 16 TiB        | 16,000 IOPS         | 1,000 MiB/s       | General workloads, flexibility in IOPS/throughput.         |
| **gp2 (General Purpose SSD)**  | 1 GiB to 16 TiB        | 16,000 IOPS         | 250 MiB/s         | General workloads, burst performance.                     |
| **io2 (Provisioned IOPS SSD)** | 4 GiB to 64 TiB        | 256,000 IOPS        | 4,000 MiB/s       | High-performance, mission-critical databases, enterprise apps. |
| **st1 (Throughput Optimized HDD)** | 125 GiB to 16 TiB | 500 IOPS            | 500 MiB/s         | Large, sequential I/O workloads like big data, log processing. |
| **sc1 (Cold HDD)**          | 125 GiB to 16 TiB      | 250 IOPS            | 250 MiB/s         | Cold storage, infrequent data access, backups, archives.   |

---

### When to Use Each Type:

- **gp3**: Ideal for most general-purpose workloads. Use when you need a balance of cost and performance and want to decouple IOPS and throughput from capacity.
  
- **gp2**: Use for general-purpose applications that occasionally require burstable performance, like small to medium-sized databases and development environments.

- **io2/io1**: Best for mission-critical, high I/O, and low-latency workloads, such as large databases and financial systems that need consistent and predictable performance.

- **st1**: Use for throughput-intensive applications like big data, data warehousing, and log processing where large amounts of data are read/written sequentially.

- **sc1**: Use for cold data storage (e.g., backups, archives) where infrequent access and cost-efficiency are more important than performance.

Choosing the right EBS volume depends on your workload requirements, including the need for IOPS, throughput, and cost considerations.

- Amazon EBS: Provides persistent block storage volumes for use with Amazon EC2 instances.
- SSD vs HDD Type Volumes: SSD volumes (gp2/gp3, io1/io2) are optimized for performance, while HDD volumes (st1, sc1) are optimized for throughput and cost.
- EBS Multi-Attach Feature: Allows a single Provisioned IOPS SSD (io1/io2) volume to be concurrently attached to multiple Nitro-based EC2 instances within the same Availability Zone.
- Copy Snapshots: Allows you to create point-in-time snapshots of your volumes and copy them across AWS regions and accounts for backup, disaster recovery, and migration purposes.
