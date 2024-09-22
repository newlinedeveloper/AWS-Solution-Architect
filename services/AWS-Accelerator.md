# AWS Global Accelerator

AWS Global Accelerator is a networking service that improves the availability and performance of your applications with global users. It provides static IP addresses that act as a fixed entry point to your application endpoints, such as Application Load Balancers (ALBs), Network Load Balancers (NLBs), and EC2 instances, in multiple AWS Regions.

## Key Features

- **Global Performance**: Routes traffic to the nearest available endpoint to reduce latency.
- **High Availability**: Automatically reroutes traffic to healthy endpoints in case of failure.
- **Static IP Addresses**: Provides two static IP addresses that serve as a fixed entry point to your application.
- **Health Checks**: Continuously monitors the health of your application endpoints.

## Connecting Multiple ALBs in Various Regions

In this example, we will use AWS Global Accelerator to connect multiple Application Load Balancers (ALBs) deployed in different AWS regions. This setup ensures that traffic is routed to the nearest and healthiest ALB, improving the performance and availability of your application.

### Example: Connecting Multiple ALBs Using AWS CDK (Go)

The following example demonstrates how to create an AWS Global Accelerator and connect it to multiple ALBs in various regions using AWS CDK in Go.

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awselasticloadbalancingv2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsglobalaccelerator"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsglobalacceleratorendpoints"
    "github.com/aws/constructs-go/constructs/v10"
)

type GlobalAcceleratorStackProps struct {
    awscdk.StackProps
}

func NewGlobalAcceleratorStack(scope constructs.Construct, id string, props *GlobalAcceleratorStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create ALB in us-west-2
    albUsWest2 := awselasticloadbalancingv2.NewApplicationLoadBalancer(stack, jsii.String("ALBUsWest2"), &awselasticloadbalancingv2.ApplicationLoadBalancerProps{
        Vpc:             vpcUsWest2, // Replace with your VPC
        InternetFacing:  jsii.Bool(true),
        LoadBalancerName: jsii.String("ALBUsWest2"),
    })

    // Create ALB in eu-central-1
    albEuCentral1 := awselasticloadbalancingv2.NewApplicationLoadBalancer(stack, jsii.String("ALBEuCentral1"), &awselasticloadbalancingv2.ApplicationLoadBalancerProps{
        Vpc:             vpcEuCentral1, // Replace with your VPC
        InternetFacing:  jsii.Bool(true),
        LoadBalancerName: jsii.String("ALBEuCentral1"),
    })

    // Create a Global Accelerator
    accelerator := awsglobalaccelerator.NewAccelerator(stack, jsii.String("GlobalAccelerator"), &awsglobalaccelerator.AcceleratorProps{
        AcceleratorName: jsii.String("MyGlobalAccelerator"),
    })

    // Add listener to the accelerator
    listener := accelerator.AddListener(jsii.String("Listener"), &awsglobalaccelerator.ListenerOptions{
        PortRanges: &[]*awsglobalaccelerator.PortRange{
            {
                FromPort: jsii.Number(80),
                ToPort:   jsii.Number(80),
            },
            {
                FromPort: jsii.Number(443),
                ToPort:   jsii.Number(443),
            },
        },
    })

    // Add endpoints to the listener
    listener.AddEndpointGroup(jsii.String("GroupUsWest2"), &awsglobalaccelerator.EndpointGroupOptions{
        Endpoints: &[]awsglobalaccelerator.IEndpoint{
            awsglobalacceleratorendpoints.NewApplicationLoadBalancerEndpoint(albUsWest2),
        },
        Region: jsii.String("us-west-2"),
    })

    listener.AddEndpointGroup(jsii.String("GroupEuCentral1"), &awsglobalaccelerator.EndpointGroupOptions{
        Endpoints: &[]awsglobalaccelerator.IEndpoint{
            awsglobalacceleratorendpoints.NewApplicationLoadBalancerEndpoint(albEuCentral1),
        },
        Region: jsii.String("eu-central-1"),
    })

    // Output ALB DNS names and Global Accelerator IPs
    awscdk.NewCfnOutput(stack, jsii.String("ALBUsWest2DNS"), &awscdk.CfnOutputProps{
        Value: albUsWest2.LoadBalancerDnsName(),
    })
    awscdk.NewCfnOutput(stack, jsii.String("ALBEuCentral1DNS"), &awscdk.CfnOutputProps{
        Value: albEuCentral1.LoadBalancerDnsName(),
    })
    awscdk.NewCfnOutput(stack, jsii.String("GlobalAcceleratorIP1"), &awscdk.CfnOutputProps{
        Value: accelerator.IpAddresses()[0],
    })
    awscdk.NewCfnOutput(stack, jsii.String("GlobalAcceleratorIP2"), &awscdk.CfnOutputProps{
        Value: accelerator.IpAddresses()[1],
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewGlobalAcceleratorStack(app, "GlobalAcceleratorStack", &GlobalAcceleratorStackProps{
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

```
# Replace with your ALB DNS names
curl -I http://<ALB_DNS_NAME_US_WEST_2>
curl -I http://<ALB_DNS_NAME_EU_CENTRAL_1>

```

```
# Replace with your Global Accelerator IP addresses
curl -I http://<GLOBAL_ACCELERATOR_IP_1>
curl -I http://<GLOBAL_ACCELERATOR_IP_2>

```

- AWS Global Accelerator: A networking service that improves the availability and performance of your applications with global users.
- Global Performance: Routes traffic to the nearest available endpoint to reduce latency.
- High Availability: Automatically reroutes traffic to healthy endpoints in case of failure.
- Static IP Addresses: Provides two static IP addresses that serve as a fixed entry point to your application.
- Health Checks: Continuously monitors the health of your application endpoints.
