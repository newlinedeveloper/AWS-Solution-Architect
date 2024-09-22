# Amazon VPC

Amazon Virtual Private Cloud (VPC) allows you to provision a logically isolated section of the AWS cloud where you can launch AWS resources in a virtual network that you define.

## Non-VPC Services

Non-VPC services are AWS services that do not require a VPC to operate. Examples include Amazon S3, Amazon DynamoDB, and Amazon SNS.

## Security Group vs NACL

### Security Group

- **Stateful**: Security groups are stateful. If you allow an incoming request from a specific IP address, the response is automatically allowed.
- **Instance-Level**: Security groups operate at the instance level.
- **Rules**: You can specify allow rules, but not deny rules.

### Network ACL (NACL)

- **Stateless**: NACLs are stateless. You must specify both inbound and outbound rules.
- **Subnet-Level**: NACLs operate at the subnet level.
- **Rules**: You can specify both allow and deny rules.

## NAT Gateways and NAT Instances

### NAT Gateway

- **Managed Service**: Fully managed by AWS.
- **High Availability**: Automatically scales and is highly available within an Availability Zone.
- **Performance**: Provides better performance and scalability compared to NAT instances.

### NAT Instance

- **Self-Managed**: You need to manage the instance yourself.
- **Single Point of Failure**: Can be a single point of failure unless you configure high availability manually.
- **Performance**: Limited by the instance type you choose.

## NAT Instance vs NAT Gateway

- **NAT Gateway**: Preferred for most use cases due to its managed nature, high availability, and better performance.
- **NAT Instance**: Can be used for custom configurations or specific use cases where you need more control.

## VPC Peering Setup

VPC peering allows you to connect two VPCs privately using AWS's network. Instances in either VPC can communicate with each other as if they are within the same network.

### Example: VPC Peering Setup

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type VpcPeeringStackProps struct {
    awscdk.StackProps
}

func NewVpcPeeringStack(scope constructs.Construct, id string, props *VpcPeeringStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create VPC 1
    vpc1 := awsec2.NewVpc(stack, jsii.String("VPC1"), &awsec2.VpcProps{
        Cidr:   jsii.String("10.0.0.0/16"),
        MaxAzs: jsii.Number(2),
    })

    // Create VPC 2
    vpc2 := awsec2.NewVpc(stack, jsii.String("VPC2"), &awsec2.VpcProps{
        Cidr:   jsii.String("10.1.0.0/16"),
        MaxAzs: jsii.Number(2),
    })

    // Create a VPC Peering Connection
    peeringConnection := awsec2.NewCfnVPCPeeringConnection(stack, jsii.String("VpcPeeringConnection"), &awsec2.CfnVPCPeeringConnectionProps{
        VpcId:        vpc1.VpcId(),
        PeerVpcId:    vpc2.VpcId(),
        PeerRegion:   jsii.String("us-west-2"),
    })

    // Update route tables to allow traffic between VPCs
    vpc1PrivateSubnets := vpc1.PrivateSubnets()
    for _, subnet := range vpc1PrivateSubnets {
        routeTable := awsec2.NewCfnRoute(stack, jsii.String("RouteToVpc2-"+*subnet.SubnetId()), &awsec2.CfnRouteProps{
            RouteTableId:         subnet.RouteTable.RouteTableId(),
            DestinationCidrBlock: jsii.String("10.1.0.0/16"),
            VpcPeeringConnectionId: peeringConnection.Ref(),
        })
    }

    vpc2PrivateSubnets := vpc2.PrivateSubnets()
    for _, subnet := range vpc2PrivateSubnets {
        routeTable := awsec2.NewCfnRoute(stack, jsii.String("RouteToVpc1-"+*subnet.SubnetId()), &awsec2.CfnRouteProps{
            RouteTableId:         subnet.RouteTable.RouteTableId(),
            DestinationCidrBlock: jsii.String("10.0.0.0/16"),
            VpcPeeringConnectionId: peeringConnection.Ref(),
        })
    }

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewVpcPeeringStack(app, "VpcPeeringStack", &VpcPeeringStackProps{
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
#### Utilizing Transit Gateway for Multi-VPC Connection
AWS Transit Gateway enables you to connect your Amazon VPCs and on-premises networks through a central hub. It simplifies network architecture and reduces the complexity of managing multiple connections.

**Example: Utilizing Transit Gateway**
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type TransitGatewayStackProps struct {
    awscdk.StackProps
}

func NewTransitGatewayStack(scope constructs.Construct, id string, props *TransitGatewayStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a Transit Gateway
    transitGateway := awsec2.NewCfnTransitGateway(stack, jsii.String("MyTransitGateway"), &awsec2.CfnTransitGatewayProps{
        Description: jsii.String("My Transit Gateway"),
    })

    // Create VPC 1
    vpc1 := awsec2.NewVpc(stack, jsii.String("VPC1"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create VPC 2
    vpc2 := awsec2.NewVpc(stack, jsii.String("VPC2"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create Transit Gateway Attachments
    awsec2.NewCfnTransitGatewayAttachment(stack, jsii.String("TGWAttachment1"), &awsec2.CfnTransitGatewayAttachmentProps{
        TransitGatewayId: transitGateway.Ref(),
        VpcId:            vpc1.VpcId(),
        SubnetIds:        vpc1.SelectSubnets(&awsec2.SubnetSelection{}).SubnetIds,
    })

    awsec2.NewCfnTransitGatewayAttachment(stack, jsii.String("TGWAttachment2"), &awsec2.CfnTransitGatewayAttachmentProps{
        TransitGatewayId: transitGateway.Ref(),
        VpcId:            vpc2.VpcId(),
        SubnetIds:        vpc2.SelectSubnets(&awsec2.SubnetSelection{}).SubnetIds,
    })

    // Output the Transit Gateway ID
    awscdk.NewCfnOutput(stack, jsii.String("TransitGatewayID"), &awscdk.CfnOutputProps{
        Value: transitGateway.Ref(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewTransitGatewayStack(app, "TransitGatewayStack", &TransitGatewayStackProps{
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

#### Adding CIDR Blocks to Your VPC
You can add additional CIDR blocks to your VPC to expand its IP address range.

**Example: Adding CIDR Blocks to Your VPC**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/constructs-go/constructs/v10"
)

type AddCidrBlockStackProps struct {
    awscdk.StackProps
}

func NewAddCidrBlockStack(scope constructs.Construct, id string, props *AddCidrBlockStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        Cidr:   jsii.String("10.0.0.0/16"),
        MaxAzs: jsii.Number(2),
    })

    // Add an additional CIDR block to the VPC
    vpc.AddCidrBlock(&awsec2.AddCidrBlockOptions{
        CidrBlock: jsii.String("10.1.0.0/16"),
    })

    // Output the VPC ID
    awscdk.NewCfnOutput(stack, jsii.String("VPCID"), &awscdk.CfnOutputProps{
        Value: vpc.VpcId(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewAddCidrBlockStack(app, "AddCidrBlockStack", &AddCidrBlockStackProps{
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

- Amazon VPC: Allows you to provision a logically isolated section of the AWS cloud.
- Non-VPC Services: AWS services that do not require a VPC to operate.
- Security Group vs NACL: Security groups are stateful and operate at the instance level, while NACLs are stateless and operate at the subnet level.
- NAT Gateways and NAT Instances: NAT gateways are managed by AWS and provide high availability, while NAT instances are self-managed.
- NAT Instance vs NAT Gateway: NAT gateways are preferred for most use cases due to their managed nature and better performance.
- VPC Peering Setup: Allows you to connect two VPCs privately.
- Utilizing Transit Gateway: Simplifies network architecture by connecting multiple VPCs through a central hub.
- Adding CIDR Blocks: Expands the IP address range of your VPC.
