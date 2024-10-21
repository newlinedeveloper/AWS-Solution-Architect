# Amazon Aurora

Amazon Aurora is a MySQL and PostgreSQL-compatible relational database built for the cloud, combining the performance and availability of high-end commercial databases with the simplicity and cost-effectiveness of open-source databases.

## Aurora Serverless Scaling

Aurora Serverless is an on-demand, auto-scaling configuration for Amazon Aurora. It automatically starts up, shuts down, and scales capacity up or down based on your application's needs.

### Example Use Case: Creating an Aurora Serverless Cluster

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsrds"
    "github.com/constructs-go/constructs/v10"
)

type AuroraServerlessStackProps struct {
    awscdk.StackProps
}

func NewAuroraServerlessStack(scope constructs.Construct, id string, props *AuroraServerlessStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an Aurora Serverless cluster
    cluster := awsrds.NewServerlessCluster(stack, jsii.String("AuroraServerlessCluster"), &awsrds.ServerlessClusterProps{
        Engine: awsrds.DatabaseClusterEngine_AuroraMysql(&awsrds.AuroraMysqlClusterEngineProps{
            Version: awsrds.AuroraMysqlEngineVersion_VER_5_7_12(),
        }),
        Vpc: vpc,
        Scaling: &awsrds.ServerlessScalingOptions{
            AutoPause: awscdk.Duration_Minutes(jsii.Number(10)),
            MinCapacity: awsrds.AuroraCapacityUnit_ACU_2(),
            MaxCapacity: awsrds.AuroraCapacityUnit_ACU_8(),
        },
    })

    // Output the cluster endpoint
    awscdk.NewCfnOutput(stack, jsii.String("ClusterEndpoint"), &awscdk.CfnOutputProps{
        Value: cluster.ClusterEndpoint().Hostname(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewAuroraServerlessStack(app, "AuroraServerlessStack", &AuroraServerlessStackProps{
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
#### High Availability for Amazon Aurora
Amazon Aurora provides high availability through multiple Availability Zones (AZs). Aurora automatically replicates your data across three AZs to provide fault tolerance and high availability.

**Example Use Case: Creating an Aurora Cluster with High Availability**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsrds"
    "github.com/constructs-go/constructs/v10"
)

type AuroraHighAvailabilityStackProps struct {
    awscdk.StackProps
}

func NewAuroraHighAvailabilityStack(scope constructs.Construct, id string, props *AuroraHighAvailabilityStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(3),
    })

    // Create an Aurora cluster with high availability
    cluster := awsrds.NewDatabaseCluster(stack, jsii.String("AuroraCluster"), &awsrds.DatabaseClusterProps{
        Engine: awsrds.DatabaseClusterEngine_AuroraMysql(&awsrds.AuroraMysqlClusterEngineProps{
            Version: awsrds.AuroraMysqlEngineVersion_VER_5_7_12(),
        }),
        Instances: jsii.Number(2),
        InstanceProps: &awsrds.InstanceProps{
            Vpc: vpc,
        },
    })

    // Output the cluster endpoint
    awscdk.NewCfnOutput(stack, jsii.String("ClusterEndpoint"), &awscdk.CfnOutputProps{
        Value: cluster.ClusterEndpoint().Hostname(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewAuroraHighAvailabilityStack(app, "AuroraHighAvailabilityStack", &AuroraHighAvailabilityStackProps{
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

#### Amazon Aurora Global Database and Replicas
Amazon Aurora Global Database allows you to create a single database that spans multiple AWS regions, enabling low-latency global reads and disaster recovery.

**Example Use Case: Creating an Aurora Global Database**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsrds"
    "github.com/constructs-go/constructs/v10"
)

type AuroraGlobalDatabaseStackProps struct {
    awscdk.StackProps
}

func NewAuroraGlobalDatabaseStack(scope constructs.Construct, id string, props *AuroraGlobalDatabaseStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an Aurora global database
    globalCluster := awsrds.NewCfnGlobalCluster(stack, jsii.String("GlobalCluster"), &awsrds.CfnGlobalClusterProps{
        GlobalClusterIdentifier: jsii.String("my-global-cluster"),
        Engine:                  jsii.String("aurora-mysql"),
        EngineVersion:           jsii.String("5.7.mysql_aurora.2.07.2"),
        StorageEncrypted:        jsii.Bool(true),
    })

    // Create a primary Aurora cluster
    primaryCluster := awsrds.NewDatabaseCluster(stack, jsii.String("PrimaryCluster"), &awsrds.DatabaseClusterProps{
        Engine: awsrds.DatabaseClusterEngine_AuroraMysql(&awsrds.AuroraMysqlClusterEngineProps{
            Version: awsrds.AuroraMysqlEngineVersion_VER_5_7_12(),
        }),
        Instances: jsii.Number(2),
        InstanceProps: &awsrds.InstanceProps{
            Vpc: vpc,
        },
        GlobalClusterIdentifier: globalCluster.GlobalClusterIdentifier(),
    })

    // Output the primary cluster endpoint
    awscdk.NewCfnOutput(stack, jsii.String("PrimaryClusterEndpoint"), &awscdk.CfnOutputProps{
        Value: primaryCluster.ClusterEndpoint().Hostname(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewAuroraGlobalDatabaseStack(app, "AuroraGlobalDatabaseStack", &AuroraGlobalDatabaseStackProps{
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

- Amazon Aurora: A MySQL and PostgreSQL-compatible relational database built for the cloud.
- Aurora Serverless Scaling: Automatically scales capacity up or down based on your application's needs.
- High Availability: Aurora replicates data across multiple Availability Zones for fault tolerance and high availability.
- Aurora Global Database: Allows you to create a single database that spans multiple AWS regions for low-latency global reads and disaster recovery.
- 

# AWS RDS vs. Amazon Aurora

## Overview

Amazon RDS (Relational Database Service) and Amazon Aurora are both managed database services provided by AWS, but they have different features, performance characteristics, and use cases. This document provides a detailed comparison of AWS RDS and Amazon Aurora, including their features, limitations, capacity details, and pricing.

## AWS RDS (Relational Database Service)

### Key Features
1. **Multiple Database Engines**:
   - Supports MySQL, PostgreSQL, MariaDB, Oracle, and Microsoft SQL Server.

2. **Automated Management**:
   - Automated backups, software patching, and database snapshots.

3. **Scalability**:
   - Vertical scaling by changing instance types.
   - Read replicas for MySQL, PostgreSQL, and MariaDB.

4. **High Availability**:
   - Multi-AZ deployments for automatic failover.

5. **Security**:
   - Encryption at rest and in transit.
   - Network isolation using Amazon VPC.

### Use Cases
- General-purpose relational databases.
- Applications requiring specific database engines like Oracle or SQL Server.
- Workloads that need automated management and high availability.

### Limitations
- Performance may not match that of Aurora for MySQL and PostgreSQL workloads.
- Limited to the features and capabilities of the underlying database engines.

### Pricing
- **Instance Pricing**: Varies based on instance type and database engine.
- **Storage Pricing**: Charged per GB-month of provisioned storage.
- **I/O Requests**: Charged per million requests for certain database engines.
- **Backup Storage**: Charged per GB-month beyond the free backup storage limit.

For detailed pricing, refer to the [AWS RDS Pricing](https://aws.amazon.com/rds/pricing/).

## Amazon Aurora

### Key Features
1. **High Performance**:
   - Up to 5 times the throughput of standard MySQL and up to 3 times the throughput of standard PostgreSQL.

2. **Scalability**:
   - Auto-scaling storage from 10 GB up to 128 TB.
   - Supports up to 15 low-latency read replicas.

3. **High Availability and Durability**:
   - Multi-AZ deployment with automatic failover.
   - Fault-tolerant storage with six-way replication across three Availability Zones.

4. **Aurora Serverless**:
   - Automatically scales compute capacity based on application demand.

5. **Compatibility**:
   - Compatible with MySQL 5.6, 5.7, and 8.0.
   - Compatible with PostgreSQL 9.6, 10, 11, 12, and 13.

### Use Cases
- High-performance MySQL and PostgreSQL applications.
- Applications requiring high availability and fault tolerance.
- Variable workloads that benefit from Aurora Serverless.

### Limitations
- Limited to MySQL and PostgreSQL compatibility.
- Higher cost compared to standard RDS for similar workloads.
- Aurora Serverless may not be suitable for all workloads, particularly those requiring consistent, high-performance compute capacity.

### Pricing
- **Instance Pricing**: Varies based on instance type.
- **Storage Pricing**: $0.10 per GB-month for storage.
- **I/O Requests**: $0.20 per million requests.
- **Backup Storage**: Charged per GB-month beyond the free backup storage limit.
- **Aurora Serverless**: Charged based on Aurora Capacity Units (ACUs) per second.

For detailed pricing, refer to the [Amazon Aurora Pricing](https://aws.amazon.com/rds/aurora/pricing/).

## Comparison Table

| Feature                    | AWS RDS                                | Amazon Aurora                          |
|----------------------------|----------------------------------------|----------------------------------------|
| **Database Engines**       | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server | MySQL, PostgreSQL                      |
| **Performance**            | Standard performance                   | High performance (5x MySQL, 3x PostgreSQL) |
| **Scalability**            | Vertical scaling, read replicas        | Auto-scaling storage, up to 15 read replicas |
| **High Availability**      | Multi-AZ deployments                   | Multi-AZ with six-way replication      |
| **Storage**                | Fixed storage, manual scaling          | Auto-scaling storage (10 GB to 128 TB) |
| **Serverless**             | Not available                          | Aurora Serverless                      |
| **Compatibility**          | Multiple database engines              | MySQL, PostgreSQL                      |
| **Cost**                   | Generally lower cost                   | Higher cost for similar workloads      |
| **Use Cases**              | General-purpose databases, specific engine requirements | High-performance, high-availability applications |

## Summary

- **AWS RDS**: A versatile managed database service supporting multiple database engines (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server). Suitable for general-purpose relational databases and applications requiring specific database engines.
- **Amazon Aurora**: A high-performance, fully managed database engine compatible with MySQL and PostgreSQL. Designed for high availability, scalability, and performance. Suitable for high-performance applications and variable workloads.

By understanding the differences between AWS RDS and Amazon Aurora, you can choose the service that best fits your specific requirements and use cases.
