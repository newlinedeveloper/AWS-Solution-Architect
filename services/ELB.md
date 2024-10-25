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

AWS provides three main types of load balancers under its **Elastic Load Balancing (ELB)** service. Each type is designed for specific use cases based on traffic patterns, application architecture, and target types. The three types of AWS load balancers are:

1. **Application Load Balancer (ALB)**  
2. **Network Load Balancer (NLB)**  
3. **Gateway Load Balancer (GWLB)**

Here’s an overview of each load balancer, their differences, and when to use them:

---

### 1. **Application Load Balancer (ALB)**
- **Layer**: Operates at **Layer 7 (Application Layer)** of the OSI model.
- **Key Features**:
  - Supports **HTTP**, **HTTPS**, and **WebSocket** protocols.
  - Can route traffic based on **content** such as URL paths, query parameters, HTTP headers, hostnames, etc. (content-based routing).
  - Supports **path-based** and **host-based** routing.
  - Provides **sticky sessions** (session affinity).
  - Can route requests to multiple services based on rules (for microservices and containers).
  - **SSL/TLS termination** (offloading SSL decryption).
  - **Integrated with AWS WAF** for security filtering.
  - **Target types**: EC2 instances, ECS tasks, Lambda functions, and IP addresses.
  - **Redirects and fixed responses** can be configured for simple actions.
- **Best For**: Web applications, microservices, HTTP/HTTPS traffic, and applications that need advanced routing (e.g., routing based on URL paths or domains).
  
- **When to Use**:
  - For **HTTP(S)** traffic or where content-based routing is needed.
  - To load balance traffic across **EC2 instances**, **containers (ECS/Fargate)**, or **Lambda** functions.
  - When you need **advanced routing rules** for microservices architecture.
  - When handling applications requiring **SSL/TLS termination**.

---

### 2. **Network Load Balancer (NLB)**
- **Layer**: Operates at **Layer 4 (Transport Layer)**.
- **Key Features**:
  - Supports **TCP**, **UDP**, and **TLS** protocols.
  - Extremely fast and capable of handling **millions of requests per second** with very low latency (~microseconds).
  - Ideal for **high-performance, low-latency** traffic.
  - Provides **static IP addresses** and can be assigned an **Elastic IP**.
  - **Connection-oriented** load balancing (maintains TCP/UDP sessions).
  - **Target types**: EC2 instances, IP addresses, and Lambda functions.
  - Can perform **TLS pass-through** or **TLS termination**.
  - Can handle **long-lived connections**, such as WebSocket or IoT applications.
  - Supports **sticky sessions** based on the source IP of the client.
  
- **Best For**: Non-HTTP traffic, performance-sensitive applications, and large volumes of TCP/UDP requests (e.g., gaming, IoT, VoIP, real-time data processing).

- **When to Use**:
  - For **TCP/UDP-based applications** where high performance and low latency are critical.
  - For **high-throughput** use cases, like **real-time gaming** or **financial transaction processing**.
  - When you need **static IP addresses** or want to assign **Elastic IPs** to the load balancer.
  - For applications requiring **long-lived connections** like WebSockets or VPNs.

---

### 3. **Gateway Load Balancer (GWLB)**
- **Layer**: Operates at **Layer 3 (Network Layer)**.
- **Key Features**:
  - Routes and distributes traffic to a fleet of virtual appliances (such as firewalls or security devices).
  - Works with **third-party network appliances** or custom virtual appliances deployed in AWS.
  - Handles **Layer 3** traffic, forwarding IP packets to virtual appliances.
  - Integrates with **AWS Firewall Manager**, **Traffic Mirroring**, and **other VPC services**.
  - Designed specifically for **traffic inspection**, **security**, and **network traffic processing** use cases.
  
- **Best For**: Deploying network appliances, such as firewalls, security, or monitoring tools across multiple VPCs.

- **When to Use**:
  - When you need to deploy **third-party virtual appliances** (e.g., firewalls, Intrusion Detection Systems) to inspect or filter traffic.
  - To route network traffic through **security appliances** before it reaches your VPC resources.

---

### Comparison of ALB vs. NLB vs. GWLB

| Feature                      | **Application Load Balancer (ALB)**        | **Network Load Balancer (NLB)**            | **Gateway Load Balancer (GWLB)**           |
|-------------------------------|--------------------------------------------|--------------------------------------------|--------------------------------------------|
| **OSI Layer**                 | Layer 7 (Application Layer)                | Layer 4 (Transport Layer)                  | Layer 3 (Network Layer)                    |
| **Protocols Supported**       | HTTP, HTTPS, WebSocket                     | TCP, UDP, TLS                              | IP (Layer 3)                               |
| **Routing**                   | Content-based (host/path/headers)          | Connection-based                           | Network-level routing                      |
| **Performance**               | High (up to millions of requests)          | Very high (millions of requests with low latency) | High                                       |
| **SSL/TLS Termination**       | Yes                                        | Yes (or TLS pass-through)                  | No                                         |
| **Sticky Sessions**           | Yes (Cookie-based)                         | Yes (Source IP-based)                      | No                                         |
| **Static IP Support**         | No                                         | Yes                                        | No                                         |
| **Use Cases**                 | Web apps, microservices, advanced routing  | Real-time apps, IoT, gaming, performance-sensitive apps | Network traffic inspection, firewalling |
| **Target Types**              | EC2, ECS, Lambda, IP addresses             | EC2, IP addresses, Lambda                  | Virtual Appliances                         |
| **Supports WebSockets**       | Yes                                        | Yes                                        | No                                         |
| **Health Checks**             | HTTP/HTTPS                                 | TCP/HTTP                                   | IP-level                                   |
| **Integration with Security** | AWS WAF                                    | N/A                                        | Virtual security appliances                |

---

### Summary: When to Use Which Load Balancer?

- **Use Application Load Balancer (ALB)**:
  - For **web applications** with HTTP/HTTPS traffic.
  - If you need **content-based routing**, such as path-based or host-based routing.
  - If you are working with **microservices** or **container-based applications** (ECS, Lambda).

- **Use Network Load Balancer (NLB)**:
  - For **high-performance**, low-latency applications using **TCP/UDP/TLS** protocols.
  - For applications that require **static IP addresses** or **Elastic IPs**.
  - For **real-time systems**, such as gaming servers, VoIP applications, or IoT services.

- **Use Gateway Load Balancer (GWLB)**:
  - If you need to route traffic through **network appliances**, such as firewalls or intrusion detection systems.
  - When building out a network security solution for multiple **VPCs**.

Each type of load balancer is optimized for specific kinds of traffic, ensuring that your application can scale effectively based on your architecture and traffic requirements.
