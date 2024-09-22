# AWS Key Management Service (KMS)

AWS Key Management Service (KMS) is a managed service that makes it easy to create and control the encryption keys used to encrypt your data. AWS KMS is integrated with other AWS services to help you protect the data you store with these services.

## AWS KMS Customer Master Key (CMK)

A Customer Master Key (CMK) is a logical key that represents the top-level encryption key in AWS KMS. CMKs are used to encrypt and decrypt data and to generate, encrypt, and decrypt data keys.

### Types of CMKs

1. **AWS Managed CMKs**: Created, managed, and used on your behalf by an AWS service integrated with AWS KMS.
2. **Customer Managed CMKs**: Created and managed by you. You have full control over these keys, including the ability to enable/disable them, define key policies, and rotate them.
3. **AWS Owned CMKs**: Used by AWS services in your account but are not visible or manageable by you.

### Example: Creating a Customer Managed CMK Using AWS CDK (Go)

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awskms"
    "github.com/constructs-go/constructs/v10"
)

type KmsStackProps struct {
    awscdk.StackProps
}

func NewKmsStack(scope constructs.Construct, id string, props *KmsStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a Customer Managed CMK
    key := awskms.NewKey(stack, jsii.String("MyKey"), &awskms.KeyProps{
        EnableKeyRotation: jsii.Bool(true),
        Alias:             jsii.String("alias/my-key"),
        Description:       jsii.String("My Customer Managed Key"),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewKmsStack(app, "KmsStack", &KmsStackProps{
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

- AWS KMS: A managed service to create and control encryption keys.
- Customer Master Key (CMK): Logical keys used to encrypt and decrypt data.
- Custom Key Store: Allows you to generate and store keys in an HSM that you control.
- Key Rotation: Periodically changing cryptographic keys to enhance security and meet compliance requirements.
