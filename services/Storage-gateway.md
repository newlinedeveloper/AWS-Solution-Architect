# AWS Storage Gateway

AWS Storage Gateway is a hybrid cloud storage service that gives you on-premises access to virtually unlimited cloud storage. It seamlessly integrates on-premises environments with the AWS storage infrastructure, providing a gateway to cloud storage for backup, archiving, and disaster recovery.

## Moving Data From AWS Storage Gateway to Amazon S3 Glacier

AWS Storage Gateway can be configured to move data to Amazon S3 Glacier for long-term, low-cost storage. This is typically done using the Tape Gateway, which provides a virtual tape library (VTL) interface that you can use with your existing backup applications.

### Example Use Case: Creating a Tape Gateway and Moving Data to Amazon S3 Glacier

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsstoragegateway"
    "github.com/constructs-go/constructs/v10"
)

type StorageGatewayToGlacierStackProps struct {
    awscdk.StackProps
}

func NewStorageGatewayToGlacierStack(scope constructs.Construct, id string, props *StorageGatewayToGlacierStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create a Tape Gateway
    tapeGateway := awsstoragegateway.NewCfnGateway(stack, jsii.String("MyTapeGateway"), &awsstoragegateway.CfnGatewayProps{
        GatewayName: jsii.String("MyTapeGateway"),
        GatewayType: jsii.String("VTL"),
        StorageGatewayRegion: jsii.String("us-west-2"),
        VpcEndpoint: jsii.String("vpce-1234567890abcdef0"),
    })

    // Create a Tape Pool for Glacier
    tapePool := awsstoragegateway.NewCfnTapePool(stack, jsii.String("MyTapePool"), &awsstoragegateway.CfnTapePoolProps{
        PoolName: jsii.String("MyGlacierPool"),
        StorageClass: jsii.String("GLACIER"),
    })

    // Output the Tape Gateway ARN
    awscdk.NewCfnOutput(stack, jsii.String("TapeGatewayARN"), &awscdk.CfnOutputProps{
        Value: tapeGateway.AttrArn(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewStorageGatewayToGlacierStack(app, "StorageGatewayToGlacierStack", &StorageGatewayToGlacierStackProps{
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
#### Integrating AWS Storage Gateway with Active Directory
AWS Storage Gateway can be integrated with Active Directory (AD) to provide seamless access to your on-premises users and applications. This is typically done using the File Gateway, which provides a file interface to Amazon S3.

**Example Use Case: Creating a File Gateway and Integrating with Active Directory**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsstoragegateway"
    "github.com/constructs-go/constructs/v10"
)

type StorageGatewayADIntegrationStackProps struct {
    awscdk.StackProps
}

func NewStorageGatewayADIntegrationStack(scope constructs.Construct, id string, props *StorageGatewayADIntegrationStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a VPC
    vpc := awsec2.NewVpc(stack, jsii.String("MyVPC"), &awsec2.VpcProps{
        MaxAzs: jsii.Number(2),
    })

    // Create a File Gateway
    fileGateway := awsstoragegateway.NewCfnGateway(stack, jsii.String("MyFileGateway"), &awsstoragegateway.CfnGatewayProps{
        GatewayName: jsii.String("MyFileGateway"),
        GatewayType: jsii.String("FILE_S3"),
        StorageGatewayRegion: jsii.String("us-west-2"),
        VpcEndpoint: jsii.String("vpce-1234567890abcdef0"),
    })

    // Integrate with Active Directory
    fileGateway.AddPropertyOverride("ActiveDirectorySettings", map[string]interface{}{
        "DomainName": jsii.String("example.com"),
        "DomainControllers": []string{
            "dc1.example.com",
            "dc2.example.com",
        },
        "UserName": jsii.String("admin"),
        "Password": jsii.String("password"),
    })

    // Output the File Gateway ARN
    awscdk.NewCfnOutput(stack, jsii.String("FileGatewayARN"), &awscdk.CfnOutputProps{
        Value: fileGateway.AttrArn(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewStorageGatewayADIntegrationStack(app, "StorageGatewayADIntegrationStack", &StorageGatewayADIntegrationStackProps{
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

- AWS Storage Gateway: A hybrid cloud storage service that integrates on-premises environments with AWS storage infrastructure.
- Moving Data to Amazon S3 Glacier: Use Tape Gateway to move data to Amazon S3 Glacier for long-term, low-cost storage.
- Integrating with Active Directory: Use File Gateway to integrate with Active Directory for seamless access to on-premises users and applications.
