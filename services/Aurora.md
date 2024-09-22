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
