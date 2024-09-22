# Amazon Elastic Kubernetes Service (EKS)

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service that makes it easy to run Kubernetes on AWS without needing to install and operate your own Kubernetes control plane or nodes.

## Creating an EKS Cluster with AWS CDK in Go

### Example Use Case: Creating an EKS Cluster

This example demonstrates how to create an EKS cluster using AWS CDK in Go.

### AWS CDK Code for Creating an EKS Cluster

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awseks"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type EKSClusterStackProps struct {
    awscdk.StackProps
}

func NewEKSClusterStack(scope constructs.Construct, id string, props *EKSClusterStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(3),
    })

    // Create an EKS cluster
    cluster := awseks.NewCluster(stack, jsii.String("MyEKSCluster"), &awseks.ClusterProps{
        Version: awseks.KubernetesVersion_V1_21(),
        Vpc: vpc,
        DefaultCapacity: jsii.Number(2),
    })

    // Output the cluster name
    awscdk.NewCfnOutput(stack, jsii.String("ClusterName"), &awscdk.CfnOutputProps{
        Value: cluster.ClusterName(),
    })

    // Output the cluster endpoint
    awscdk.NewCfnOutput(stack, jsii.String("ClusterEndpoint"), &awscdk.CfnOutputProps{
        Value: cluster.ClusterEndpoint(),
    })

    // Output the cluster ARN
    awscdk.NewCfnOutput(stack, jsii.String("ClusterARN"), &awscdk.CfnOutputProps{
        Value: cluster.ClusterArn(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewEKSClusterStack(app, "EKSClusterStack", &EKSClusterStackProps{
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
- Amazon EKS: A managed Kubernetes service that makes it easy to run Kubernetes on AWS.
- Creating an EKS Cluster: Use AWS CDK in Go to create an EKS cluster with a VPC and default node capacity.
