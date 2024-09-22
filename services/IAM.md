# AWS Identity and Access Management (IAM)

AWS Identity and Access Management (IAM) enables you to manage access to AWS services and resources securely. Using IAM, you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.

## Identity-based Policies and Resource-based Policies

### Identity-based Policies

Identity-based policies are JSON permissions policies that you attach to IAM identities (users, groups, and roles). These policies define what actions an identity can perform on which resources, and under what conditions.

#### Example: Identity-based Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::example-bucket"
    },
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}

```
#### Resource-based Policies
Resource-based policies are JSON permissions policies that you attach to resources such as Amazon S3 buckets, Amazon SQS queues, and AWS KMS keys. These policies define what actions can be performed on the resource, by which principals, and under what conditions.

**Example: Resource-based Policy**
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/Alice"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

#### IAM Permissions Boundary
A permissions boundary is an advanced feature for using a managed policy to set the maximum permissions that an IAM entity (user or role) can have. Permissions boundaries are used to ensure that users or roles do not exceed the permissions defined in the boundary.

**Example: Permissions Boundary Policy**

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteBucket",
      "Resource": "*"
    }
  ]
}

```

IAM Policy Structure and Conditions

- Effect: Specifies whether the statement results in an allow or deny.
- Action: Specifies the actions that are allowed or denied.
- Resource: Specifies the resources to which the actions apply.
- Condition: (Optional) Specifies conditions under which the statement is in effect.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::example-bucket/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "public-read"
        }
      }
    }
  ]
}

```

- Identity-based Policies: Attached to IAM identities (users, groups, roles) to define permissions.
- Resource-based Policies: Attached to resources to define who can access them and what actions they can perform.
- Permissions Boundary: Managed policy that sets the maximum permissions an IAM entity can have.
- Policy Structure: Consists of statements with elements like Effect, Action, Resource, and Condition.
- Policy Evaluation Logic: Follows the order of explicit deny, explicit allow, and default deny.

# AWS Identity and Access Management (IAM)

AWS Identity and Access Management (IAM) enables you to manage access to AWS services and resources securely. IAM provides several key components and features to help you manage permissions and access control.

## Key Components and Features in IAM

### 1. IAM Users

IAM users are individuals or applications that need access to AWS resources. Each user has a unique identity and can have its own set of credentials.

### 2. IAM Groups

IAM groups are collections of IAM users. You can attach policies to groups, and all users in the group inherit the permissions specified in the group policies.

### 3. IAM Roles

IAM roles are identities that you can assume to gain temporary access to AWS resources. Roles are useful for granting permissions to AWS services, applications, or users from other AWS accounts.

### 4. IAM Policies

IAM policies are JSON documents that define permissions. Policies can be attached to users, groups, or roles to specify what actions they can perform on which resources.

### 5. Multi-Factor Authentication (MFA)

MFA adds an extra layer of security by requiring users to provide a second form of authentication (e.g., a code from a hardware or virtual MFA device) in addition to their password.

### 6. Access Analyzer

IAM Access Analyzer helps you identify resources in your account that are shared with an external entity. It provides insights into access permissions and helps you ensure that your resources are not unintentionally exposed.

### 7. Service Control Policies (SCPs)

SCPs are policies that you can use to manage permissions in AWS Organizations. SCPs allow you to set permission guardrails for all accounts in an organization, ensuring that accounts cannot exceed the permissions defined in the SCPs.

## Real-World Use Cases

### Use Case 1: Creating IAM Users and Groups

In this use case, we create IAM users and groups using AWS CDK in Go. We attach policies to the group to grant permissions to the users in the group.

### Example: Creating IAM Users and Groups Using AWS CDK (Go)

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsiam"
    "github.com/aws/constructs-go/constructs/v10"
)

type IamStackProps struct {
    awscdk.StackProps
}

func NewIamStack(scope constructs.Construct, id string, props *IamStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an IAM group
    group := awsiam.NewGroup(stack, jsii.String("MyGroup"), &awsiam.GroupProps{
        GroupName: jsii.String("Developers"),
    })

    // Attach a policy to the group
    group.AddManagedPolicy(awsiam.ManagedPolicy_FromAwsManagedPolicyName(jsii.String("AmazonS3ReadOnlyAccess")))

    // Create IAM users and add them to the group
    user1 := awsiam.NewUser(stack, jsii.String("User1"), &awsiam.UserProps{
        UserName: jsii.String("Alice"),
    })
    group.AddUser(user1)

    user2 := awsiam.NewUser(stack, jsii.String("User2"), &awsiam.UserProps{
        UserName: jsii.String("Bob"),
    })
    group.AddUser(user2)

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewIamStack(app, "IamStack", &IamStackProps{
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
**Example: Creating IAM Roles for Cross-Account Access Using AWS CDK (Go)**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsiam"
    "github.com/aws/constructs-go/constructs/v10"
)

type CrossAccountRoleStackProps struct {
    awscdk.StackProps
}

func NewCrossAccountRoleStack(scope constructs.Construct, id string, props *CrossAccountRoleStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an IAM role for cross-account access
    role := awsiam.NewRole(stack, jsii.String("CrossAccountRole"), &awsiam.RoleProps{
        AssumedBy: awsiam.NewAccountPrincipal(jsii.String("123456789012")), // Replace with the AWS account ID of the trusted account
        RoleName:  jsii.String("CrossAccountAccessRole"),
    })

    // Attach a policy to the role
    role.AddManagedPolicy(awsiam.ManagedPolicy_FromAwsManagedPolicyName(jsii.String("AmazonS3ReadOnlyAccess")))

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewCrossAccountRoleStack(app, "CrossAccountRoleStack", &CrossAccountRoleStackProps{
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
