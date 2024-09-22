# AWS Elastic Load Balancing (ELB)

AWS Elastic Load Balancing (ELB) automatically distributes incoming application traffic across multiple targets, such as Amazon EC2 instances, containers, and IP addresses, in one or more Availability Zones. ELB helps ensure the availability and scalability of your applications.

## Components of AWS Elastic Load Balancing

1. **Load Balancer**: The entry point for your application traffic. ELB offers different types of load balancers to handle different types of traffic.
2. **Listeners**: A process that checks for connection requests from clients using the configured protocol and port.
3. **Target Groups**: A group of targets (e.g., EC2 instances) that receive traffic from the load balancer.
4. **Rules**: Conditions that define how the load balancer routes requests to the target groups.
5. **Health Checks**: Checks performed by the load balancer to ensure that the targets are healthy and can handle traffic.

## AWS ELB Request Routing Algorithms

ELB uses different routing algorithms to distribute traffic among targets:

1. **Round Robin**: Distributes requests evenly across all targets in the target group. Used by Application Load Balancers and Classic Load Balancers.
2. **Least Outstanding Requests**: Routes requests to the target with the fewest outstanding requests. Used by Application Load Balancers.
3. **Flow Hash**: Routes requests based on a hash of the source and destination IP addresses and ports. Used by Network Load Balancers.

## ELB Idle Timeout

The ELB idle timeout is the amount of time that a connection is allowed to be idle before it is closed by the load balancer. The default idle timeout is 60 seconds, but it can be configured to a value between 1 and 4000 seconds.

### Example: Configuring Idle Timeout

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awselasticloadbalancingv2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type ElbIdleTimeoutStackProps struct {
    awscdk.StackProps
}

func NewElbIdleTimeoutStack(scope constructs.Construct, id string, props *ElbIdleTimeoutStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an Application Load Balancer
    alb := awselasticloadbalancingv2.NewApplicationLoadBalancer(stack, jsii.String("MyALB"), &awselasticloadbalancingv2.ApplicationLoadBalancerProps{
        Vpc:            vpc,
        InternetFacing: jsii.Bool(true),
    })

    // Configure idle timeout
    alb.SetIdleTimeout(awscdk.Duration_Seconds(jsii.Number(120)))

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewElbIdleTimeoutStack(app, "ElbIdleTimeoutStack", &ElbIdleTimeoutStackProps{
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

#### ELB Health Checks vs Route 53 Health Checks for Target Health Monitoring
**ELB Health Checks**
- Performed by: The load balancer.
- Purpose: To determine the health of targets (e.g., EC2 instances) and route traffic only to healthy targets.
- Configuration: Can be configured with parameters such as interval, timeout, healthy threshold, and unhealthy threshold.

**Route 53 Health Checks**
- Performed by: Amazon Route 53.
- Purpose: To monitor the health of endpoints (e.g., web servers) and route DNS traffic based on health status.
- Configuration: Can be configured with parameters such as interval, timeout, and failure threshold.

#### Application Load Balancer vs Network Load Balancer vs Classic Load Balancer vs Gateway Load Balancer

**Application Load Balancer (ALB)**
- Layer: Operates at the application layer (Layer 7).
- Use Case: Ideal for HTTP/HTTPS traffic and advanced routing based on content.
- Features: Supports host-based and path-based routing, WebSocket, and HTTP/2.

**Network Load Balancer (NLB)**
- Layer: Operates at the transport layer (Layer 4).
- Use Case: Ideal for TCP/UDP traffic and high-performance, low-latency applications.
- Features: Supports static IP addresses, preserves the source IP address, and handles millions of requests per second.

**Classic Load Balancer (CLB)**
- Layer: Operates at both the application layer (Layer 7) and transport layer (Layer 4).
- Use Case: Legacy applications and simple load balancing needs.
- Features: Supports basic load balancing for HTTP/HTTPS and TCP traffic.

**Gateway Load Balancer (GWLB)**
- Layer: Operates at the network layer (Layer 3).
- Use Case: Ideal for deploying, scaling, and managing third-party virtual appliances such as firewalls, intrusion detection and prevention systems, and deep packet inspection systems.
- Features: Integrates with AWS Gateway Load Balancer endpoints (GWLBe) for seamless traffic routing.

#### Application Load Balancer Listener Rule Conditions
Listener rules for an Application Load Balancer define how requests are routed to target groups based on conditions. Common conditions include:

- Host Header: Routes requests based on the host header (e.g., example.com).
- Path: Routes requests based on the URL path (e.g., /images).
- HTTP Header: Routes requests based on HTTP headers (e.g., User-Agent).
- HTTP Method: Routes requests based on HTTP methods (e.g., GET, POST).
- Query String: Routes requests based on query string parameters (e.g., ?user=123).
- Source IP: Routes requests based on the source IP address.

**Example: Configuring Listener Rule Conditions**
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awselasticloadbalancingv2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type AlbListenerRulesStackProps struct {
    awscdk.StackProps
}

func NewAlbListenerRulesStack(scope constructs.Construct, id string, props *AlbListenerRulesStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create an Application Load Balancer
    alb := awselasticloadbalancingv2.NewApplicationLoadBalancer(stack, jsii.String("MyALB"), &awselasticloadbalancingv2.ApplicationLoadBalancerProps{
        Vpc:            vpc,
        InternetFacing: jsii.Bool(true),
    })

    // Create a listener
    listener := alb.AddListener(jsii.String("Listener"), &awselasticloadbalancingv2.BaseApplicationListenerProps{
        Port: jsii.Number(80),
    })

    // Create target groups
    targetGroup1 := awselasticloadbalancingv2.NewApplicationTargetGroup(stack, jsii.String("TargetGroup1"), &awselasticloadbalancingv2.ApplicationTargetGroupProps{
        Vpc:     vpc,
        Port:    jsii.Number(80),
        Protocol: awselasticloadbalancingv2.ApplicationProtocol_HTTP,
        Targets: &[]awselasticloadbalancingv2.IApplicationLoadBalancerTarget{},
    })

    targetGroup2 := awselasticloadbalancingv2.NewApplicationTargetGroup(stack, jsii.String("TargetGroup2"), &awselasticloadbalancingv2.ApplicationTargetGroupProps{
        Vpc:     vpc,
        Port:    jsii.Number(80),
        Protocol: awselasticloadbalancingv2.ApplicationProtocol_HTTP,
        Targets: &[]awselasticloadbalancingv2.IApplicationLoadBalancerTarget{},
    })

    // Add listener rules
    listener.AddTargetGroups(jsii.String("DefaultRule"), &awselasticloadbalancingv2.AddApplicationTargetGroupsProps{
        TargetGroups: &[]awselasticloadbalancingv2.IApplicationTargetGroup{targetGroup1},
    })

    listener.AddTargetGroups(jsii.String("HostHeaderRule"), &awselasticloadbalancingv2.AddApplicationTargetGroupsProps{
        Conditions: &[]awselasticloadbalancingv2.ListenerCondition{
            awselasticloadbalancingv2.ListenerCondition_HostHeaders(&[]*string{jsii.String("example.com")}),
        },
        TargetGroups: &[]awselasticloadbalancingv2.IApplicationTargetGroup{targetGroup2},
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewAlbListenerRulesStack(app, "AlbListenerRulesStack", &AlbListenerRulesStackProps{
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

- AWS Elastic Load Balancing (ELB): Automatically distributes incoming application traffic across multiple targets to ensure availability and scalability.
- Components: Load balancer, listeners, target groups, rules, and health checks.
- Request Routing Algorithms: Round Robin, Least Outstanding Requests, and Flow Hash.
- ELB Idle Timeout: Configurable idle timeout for connections.
- Health Checks: ELB health checks vs. Route 53 health checks.
- Types of Load Balancers: Application Load Balancer, Network Load Balancer, Classic Load Balancer, and Gateway Load Balancer.
- Listener Rule Conditions: Host header, path, HTTP header, HTTP method, query string, and source IP.

