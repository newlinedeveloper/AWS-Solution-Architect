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

AWS provides a variety of policy types to manage permissions for users, services, and resources. These policies allow you to define what actions are allowed or denied on AWS resources and are essential for managing security and access control. Below is an explanation of each policy type, along with examples and use cases for each:

---

### 1. **Identity-Based Policies**
- **Definition**: Identity-based policies are attached to **IAM identities** such as users, groups, or roles. They define what actions the identity can perform on which resources.
- **Common Types**:
  - **Managed Policies**: Predefined policies provided by AWS or created by you that can be attached to multiple identities.
  - **Inline Policies**: Policies that are directly attached to a specific user, group, or role.

#### Example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::example-bucket"
    }
  ]
}
```
This identity-based policy allows the IAM user or role to list objects in the S3 bucket `example-bucket`.

#### Use Case:
- **When to Use**: Use identity-based policies when you want to define what **actions** a particular **user, group, or role** can perform on AWS resources.
- **Scenario**: You want to allow an IAM user or role to access certain resources like S3, EC2, or RDS.

---

### 2. **Resource-Based Policies**
- **Definition**: Resource-based policies are attached directly to AWS resources (such as S3 buckets, SNS topics, and SQS queues) and define who can access that resource and what actions they can perform.
- **Commonly Used With**: S3 buckets, SNS topics, SQS queues, Lambda functions.

#### Example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/S3AccessRole"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```
This resource-based policy allows the role `S3AccessRole` to retrieve objects from the S3 bucket `example-bucket`.

#### Use Case:
- **When to Use**: Use resource-based policies when you want to define which **users, roles, or AWS services** can interact with a specific **resource**.
- **Scenario**: You want to grant access to an S3 bucket to users from another AWS account or allow a Lambda function to access an SQS queue.

---

### 3. **Permissions Boundaries**
- **Definition**: Permissions boundaries are an advanced feature for managing permissions. They define the **maximum permissions** that an IAM identity (user or role) can have. These boundaries act as a guardrail, ensuring that users or roles cannot exceed the defined permissions, even if other policies allow broader permissions.
- **Common Use**: Used in environments where delegated administration is needed to limit the permissions that can be granted by other admins.

#### Example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::example-bucket"
    }
  ]
}
```
This permissions boundary restricts the IAM identity to only access the S3 bucket `example-bucket` and prevents granting broader permissions.

#### Use Case:
- **When to Use**: Use permissions boundaries to set limits on the permissions that IAM users or roles can have.
- **Scenario**: You allow team leads to create roles for their team, but you want to ensure they cannot grant permissions beyond a certain boundary (e.g., no access to critical resources like RDS or DynamoDB).

---

### 4. **AWS Organizations Service Control Policies (SCPs)**
- **Definition**: SCPs are used within AWS Organizations to define the **maximum permissions** that can be applied to the accounts in your organization. SCPs do not grant permissions by themselves; they act as guardrails by limiting the maximum permissions that can be granted by identity-based policies.
- **Scope**: SCPs apply to all IAM identities (users, groups, roles) within an AWS account.

#### Example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "ec2:TerminateInstances",
      "Resource": "*"
    }
  ]
}
```
This SCP denies all users in the organization the ability to terminate EC2 instances, regardless of their individual permissions.

#### Use Case:
- **When to Use**: Use SCPs to enforce organizational policies across multiple AWS accounts.
- **Scenario**: You want to ensure that no one in your organization can create certain high-cost resources (e.g., EC2 instances with a specific instance type) or perform sensitive actions (e.g., deleting RDS databases).

---

### 5. **Access Control Lists (ACLs)**
- **Definition**: ACLs are used to control access to specific AWS resources at a more granular level. Unlike IAM policies, which are JSON-based, ACLs are more limited in functionality and are often used for **network-level** access control.
- **Common Use**: Primarily used with Amazon S3 and VPC to control network-level access.

#### Example (S3 Bucket ACL):
```json
{
  "Grants": [
    {
      "Grantee": {
        "ID": "CanonicalUserID",
        "Type": "CanonicalUser"
      },
      "Permission": "READ"
    }
  ]
}
```
This ACL allows the specified user to have read access to the S3 bucket.

#### Use Case:
- **When to Use**: Use ACLs when you need to provide **basic access control** at the **resource level**.
- **Scenario**: You want to control access to an S3 bucket by granting **read-only access** to specific users or applications without using an IAM policy.

---

### 6. **Session Policies**
- **Definition**: Session policies are policies that are passed when an **IAM role** or **federated user** assumes a role via `sts:AssumeRole`. These policies further restrict the permissions granted by the role's identity-based policies and are temporary in nature.
- **Common Use**: Used in scenarios where temporary permissions need to be defined or constrained.

#### Example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```
This session policy allows a federated user to retrieve objects from the S3 bucket `example-bucket` during their session.

#### Use Case:
- **When to Use**: Use session policies when you need to apply temporary restrictions during an IAM role session.
- **Scenario**: A user assumes a role for a limited period, and you want to restrict their permissions to only specific actions (e.g., read-only access to S3 objects).

---

### When to Use Each Policy Type:

| **Policy Type**           | **Use Case**                                                                                                                                   |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| **Identity-Based Policies**   | Attach policies to IAM users, groups, or roles to define what actions they can perform. Suitable for day-to-day access control.               |
| **Resource-Based Policies**   | Control who can access a specific AWS resource. Ideal when you want to give cross-account access or allow services (e.g., Lambda) to access resources. |
| **Permissions Boundaries**    | Limit the permissions that an IAM role or user can have. Useful for delegated administration and limiting what permissions can be granted.      |
| **SCPs (AWS Organizations)** | Enforce guardrails for all accounts in an AWS Organization, ensuring that no user or role can exceed specific permissions.                     |
| **ACLs**                     | Basic access control for specific AWS services like S3 or VPC at a network level. Suitable for simple permission models.                      |
| **Session Policies**         | Restrict the permissions of a temporary session when a role is assumed. Ideal for granting temporary, limited permissions to users or services. |

---

These policies offer flexible and comprehensive ways to manage access and permissions in AWS, enabling you to ensure security and compliance across your AWS environment.

An **assumed role** in AWS Identity and Access Management (IAM) refers to a role that a user or service can temporarily "assume" to obtain a set of permissions that are different from their usual IAM identity. This is typically done to give limited or cross-account access, enhance security, or apply the principle of least privilege.

### Key Concepts:
- **IAM Role**: A set of permissions without being associated with a specific user or group. It can be assumed by anyone (or any service) authorized to do so.
- **Assuming a Role**: When an IAM user, AWS service, or external identity assumes a role, they temporarily inherit the permissions defined by the role's policies.

### Use Cases for Assumed Roles
1. **Cross-Account Access**: A role in one AWS account can be assumed by users or services in another account.
2. **Temporary Elevated Permissions**: A user can assume a role with higher permissions for a limited period (e.g., for administrative tasks).
3. **Service Roles**: AWS services like EC2, Lambda, or ECS assume roles to perform actions on your behalf.
4. **Delegation of Access**: A role can be assumed by an external identity (e.g., federated users, third-party systems).

---

### How Role Assumption Works:
1. **Create an IAM Role**: The role defines a set of permissions via its policy.
2. **Specify the Trust Relationship**: A trust policy is attached to the role, defining who or what can assume the role.
3. **Assume the Role**: A user, service, or entity assumes the role using the `sts:AssumeRole` API, receiving temporary security credentials (access key, secret key, session token) to use the role's permissions.

---

### Example: Cross-Account Access with Assumed Role

#### Scenario:
You have **Account A** (account ID: `111111111111`) and **Account B** (account ID: `222222222222`). You want users in Account B to assume a role in Account A and access an S3 bucket in Account A.

#### Step-by-Step Configuration:

1. **Create an IAM Role in Account A**:
   In Account A, create a role that allows access to the S3 bucket.

   **Trust Policy** (who can assume the role):
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::222222222222:root"  // Account B
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

   This trust policy allows users from Account B to assume this role.

2. **Attach Permissions to the Role**:
   Attach a policy to the role that grants the necessary permissions, such as access to an S3 bucket.

   **Permissions Policy** (what the role can do):
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": "s3:*",
         "Resource": "arn:aws:s3:::my-bucket/*"
       }
     ]
   }
   ```

   This policy allows the role to perform all actions on the `my-bucket` S3 bucket in Account A.

3. **Assume the Role from Account B**:
   In Account B, a user can assume the role in Account A using the AWS CLI, SDK, or API.

   **Example using AWS CLI**:
   ```bash
   aws sts assume-role \
     --role-arn arn:aws:iam::111111111111:role/S3AccessRole \
     --role-session-name CrossAccountAccessSession
   ```

   This command will return temporary security credentials (access key, secret key, and session token) that allow the user to perform the actions specified in the role’s policy.

4. **Use Temporary Credentials**:
   After assuming the role, the user can use the temporary credentials to interact with the S3 bucket in Account A.

   **Example CLI Command with Temporary Credentials**:
   ```bash
   aws s3 ls s3://my-bucket/ \
     --access-key <temporary-access-key> \
     --secret-key <temporary-secret-key> \
     --session-token <temporary-session-token>
   ```

---

### Example: Temporary Elevated Permissions with Assumed Role

#### Scenario:
You want a user to perform administrative tasks (e.g., manage EC2 instances) only when they explicitly assume an **AdminRole**.

1. **Create an Admin Role**:
   Create a role in the same AWS account and attach a policy that allows administrative actions.

   **Admin Role Permissions**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": "ec2:*",
         "Resource": "*"
       }
     ]
   }
   ```

   This policy allows full access to EC2.

2. **Trust Policy** (to allow only specific users to assume the role):
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::111111111111:user/john"
         },
         "Action": "sts:AssumeRole"
       }
     ]
   }
   ```

3. **Assuming the Admin Role**:
   The user (John) must explicitly assume the **AdminRole** to gain the temporary elevated permissions.

   **Example using AWS CLI**:
   ```bash
   aws sts assume-role \
     --role-arn arn:aws:iam::111111111111:role/AdminRole \
     --role-session-name AdminSession
   ```

   John can now use the temporary credentials to perform EC2 actions within the time-limited session.

---

### Key Points about Assumed Roles:
- **Temporary Access**: When you assume a role, you get temporary security credentials (typically valid for 15 minutes to several hours) that expire after the session.
- **Least Privilege**: Assumed roles promote the principle of least privilege, as users only get permissions for a limited period when needed.
- **Cross-Account**: Roles are often used to facilitate secure cross-account access between different AWS accounts.
- **Delegation of Access**: Roles can be used to delegate access to AWS resources without sharing long-term credentials like IAM user access keys.

---

### Summary:
An **assumed role** in AWS is a powerful mechanism to grant temporary, scoped access to resources either within the same AWS account or across different AWS accounts. Whether it's used for cross-account access, temporary elevated permissions, or for AWS services to interact with other resources, it helps maintain a secure, flexible permission model.
