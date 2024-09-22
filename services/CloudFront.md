# Amazon CloudFront

Amazon CloudFront is a fast content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally with low latency and high transfer speeds. CloudFront integrates with other AWS services to provide a comprehensive solution for delivering content.

## Key Components of Amazon CloudFront

1. **Distributions**: A distribution is the configuration entity within CloudFront that contains information about where you want your content to be delivered from and the details about how to track and manage content delivery.
2. **Origins**: The origin is the source of the content that CloudFront will distribute. It can be an Amazon S3 bucket, an HTTP server, or an AWS MediaPackage channel.
3. **Edge Locations**: These are the locations where CloudFront caches copies of your content for faster delivery to users.
4. **Behaviors**: Behaviors define how CloudFront will handle requests for your content. You can configure multiple behaviors for a single distribution.
5. **Origin Access Identity (OAI)**: A special CloudFront user that you can associate with your distributions to restrict access to content in Amazon S3 buckets.

## Custom DNS Names with Dedicated SSL Certificates for Your CloudFront Distribution

You can use custom DNS names (CNAMEs) and dedicated SSL certificates to serve your content securely over HTTPS.

### Example: Setting Up Custom DNS Names with Dedicated SSL Certificates

1. **Create a CloudFront Distribution**: Create a CloudFront distribution and specify your origin.
2. **Add Alternate Domain Names (CNAMEs)**: Add your custom domain names to the distribution.
3. **Request an SSL Certificate**: Use AWS Certificate Manager (ACM) to request an SSL certificate for your custom domain.
4. **Associate the SSL Certificate**: Associate the SSL certificate with your CloudFront distribution.

### Example: AWS CDK Code for Custom DNS Names with SSL Certificates

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awscertificatemanager"
    "github.com/aws/aws-cdk-go/awscdk/v2/awscdkcloudfront"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
    "github.com/constructs-go/constructs/v10"
)

type CloudFrontStackProps struct {
    awscdk.StackProps
}

func NewCloudFrontStack(scope constructs.Construct, id string, props *CloudFrontStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an S3 bucket to serve as the origin
    bucket := awss3.NewBucket(stack, jsii.String("MyBucket"), &awss3.BucketProps{})

    // Request an SSL certificate for the custom domain
    certificate := awscertificatemanager.NewCertificate(stack, jsii.String("MyCertificate"), &awscertificatemanager.CertificateProps{
        DomainName: jsii.String("example.com"),
        Validation: awscertificatemanager.CertificateValidation_FromDns(),
    })

    // Create a CloudFront distribution with custom DNS names and SSL certificate
    distribution := awscdkcloudfront.NewCloudFrontWebDistribution(stack, jsii.String("MyDistribution"), &awscdkcloudfront.CloudFrontWebDistributionProps{
        OriginConfigs: &[]*awscdkcloudfront.SourceConfiguration{
            {
                S3OriginSource: &awscdkcloudfront.S3OriginConfig{
                    S3BucketSource: bucket,
                },
                Behaviors: &[]*awscdkcloudfront.Behavior{
                    {
                        IsDefaultBehavior: jsii.Bool(true),
                    },
                },
            },
        },
        AliasConfiguration: &awscdkcloudfront.AliasConfiguration{
            AcmsCertificateArn: certificate.CertificateArn(),
            Names:              &[]*string{jsii.String("example.com")},
        },
    })

    // Output the CloudFront distribution domain name
    awscdk.NewCfnOutput(stack, jsii.String("CloudFrontDomainName"), &awscdk.CfnOutputProps{
        Value: distribution.DistributionDomainName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewCloudFrontStack(app, "CloudFrontStack", &CloudFrontStackProps{
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

### Restricting Content Access with Signed URLs and Signed Cookies
You can restrict access to your content by using signed URLs and signed cookies. This ensures that only authorized users can access your content.

Example: Restricting Access with Signed URLs
- Create a CloudFront Key Pair: Create a key pair in the AWS Management Console.
- Generate Signed URLs: Use the private key to generate signed URLs for your content.
- Distribute Signed URLs: Distribute the signed URLs to authorized users.
**Example: AWS CDK Code for Signed URLs**
```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awscdkcloudfront"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
    "github.com/constructs-go/constructs/v10"
)

type CloudFrontSignedUrlsStackProps struct {
    awscdk.StackProps
}

func NewCloudFrontSignedUrlsStack(scope constructs.Construct, id string, props *CloudFrontSignedUrlsStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an S3 bucket to serve as the origin
    bucket := awss3.NewBucket(stack, jsii.String("MyBucket"), &awss3.BucketProps{})

    // Create a CloudFront distribution with signed URLs
    distribution := awscdkcloudfront.NewCloudFrontWebDistribution(stack, jsii.String("MyDistribution"), &awscdkcloudfront.CloudFrontWebDistributionProps{
        OriginConfigs: &[]*awscdkcloudfront.SourceConfiguration{
            {
                S3OriginSource: &awscdkcloudfront.S3OriginConfig{
                    S3BucketSource: bucket,
                },
                Behaviors: &[]*awscdkcloudfront.Behavior{
                    {
                        IsDefaultBehavior: jsii.Bool(true),
                    },
                },
            },
        },
    })

    // Output the CloudFront distribution domain name
    awscdk.NewCfnOutput(stack, jsii.String("CloudFrontDomainName"), &awscdk.CfnOutputProps{
        Value: distribution.DistributionDomainName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewCloudFrontSignedUrlsStack(app, "CloudFrontSignedUrlsStack", &CloudFrontSignedUrlsStackProps{
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

#### Origin Access Identity in CloudFront
An Origin Access Identity (OAI) is a special CloudFront user that you can associate with your distributions to restrict access to content in Amazon S3 buckets.

Example: Setting Up Origin Access Identity
- Create an OAI: Create an OAI in the AWS Management Console.
- Associate OAI with Distribution: Associate the OAI with your CloudFront distribution.
- Update S3 Bucket Policy: Update the S3 bucket policy to grant access to the OAI.
**Example: AWS CDK Code for Origin Access Identity**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awscdkcloudfront"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
    "github.com/constructs-go/constructs/v10"
)

type CloudFrontOAIStackProps struct {
    awscdk.StackProps
}

func NewCloudFrontOAIStack(scope constructs.Construct, id string, props *CloudFrontOAIStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an S3 bucket to serve as the origin
    bucket := awss3.NewBucket(stack, jsii.String("MyBucket"), &awss3.BucketProps{})

    // Create an Origin Access Identity (OAI)
    oai := awscdkcloudfront.NewOriginAccessIdentity(stack, jsii.String("MyOAI"), &awscdkcloudfront.OriginAccessIdentityProps{})

    // Grant read permissions to the OAI
    bucket.GrantRead(oai.GrantPrincipal())

    // Create a CloudFront distribution with OAI
    distribution := awscdkcloudfront.NewCloudFrontWebDistribution(stack, jsii.String("MyDistribution"), &awscdkcloudfront.CloudFrontWebDistributionProps{
        OriginConfigs: &[]*awscdkcloudfront.SourceConfiguration{
            {
                S3OriginSource: &awscdkcloudfront.S3OriginConfig{
                    S3BucketSource: bucket,
                    OriginAccessIdentity: oai,
                },
                Behaviors: &[]*awscdkcloudfront.Behavior{
                    {
                        IsDefaultBehavior: jsii.Bool(true),
                    },
                },
            },
        },
    })

    // Output the CloudFront distribution domain name
    awscdk.NewCfnOutput(stack, jsii.String("CloudFrontDomainName"), &awscdk.CfnOutputProps{
        Value: distribution.DistributionDomainName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewCloudFrontOAIStack(app, "CloudFrontOAIStack", &CloudFrontOAIStackProps{
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

#### High Availability with CloudFront Origin Failover
CloudFront can be configured to use origin failover to improve the availability of your application. If the primary origin fails, CloudFront automatically switches to a secondary origin.

Example: Setting Up Origin Failover
- Create Primary and Secondary Origins: Set up your primary and secondary origins.
- Configure Origin Groups: Configure an origin group in CloudFront to include both origins.
- Set Up Failover Criteria: Define the failover criteria based on HTTP status codes or other conditions.
**Example: AWS CDK Code for Origin Failover**

```
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awscdkcloudfront"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
    "github.com/constructs-go/constructs/v10"
)

type CloudFrontFailoverStackProps struct {
    awscdk.StackProps
}

func NewCloudFrontFailoverStack(scope constructs.Construct, id string, props *CloudFrontFailoverStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create primary and secondary S3 buckets to serve as origins
    primaryBucket := awss3.NewBucket(stack, jsii.String("PrimaryBucket"), &awss3.BucketProps{})
    secondaryBucket := awss3.NewBucket(stack, jsii.String("SecondaryBucket"), &awss3.BucketProps{})

    // Create a CloudFront distribution with origin failover
    distribution := awscdkcloudfront.NewCloudFrontWebDistribution(stack, jsii.String("MyDistribution"), &awscdkcloudfront.CloudFrontWebDistributionProps{
        OriginConfigs: &[]*awscdkcloudfront.SourceConfiguration{
            {
                S3OriginSource: &awscdkcloudfront.S3OriginConfig{
                    S3BucketSource: primaryBucket,
                },
                Behaviors: &[]*awscdkcloudfront.Behavior{
                    {
                        IsDefaultBehavior: jsii.Bool(true),
                    },
                },
            },
            {
                S3OriginSource: &awscdkcloudfront.S3OriginConfig{
                    S3BucketSource: secondaryBucket,
                },
                FailoverCriteriaStatusCodes: &[]*float64{
                    jsii.Number(500),
                    jsii.Number(502),
                    jsii.Number(503),
                    jsii.Number(504),
                },
                Behaviors: &[]*awscdkcloudfront.Behavior{
                    {
                        IsDefaultBehavior: jsii.Bool(false),
                    },
                },
            },
        },
    })

    // Output the CloudFront distribution domain name
    awscdk.NewCfnOutput(stack, jsii.String("CloudFrontDomainName"), &awscdk.CfnOutputProps{
        Value: distribution.DistributionDomainName(),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewCloudFrontFailoverStack(app, "CloudFrontFailoverStack", &CloudFrontFailoverStackProps{
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

- Amazon CloudFront: A fast content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally with low latency and high transfer speeds.
- Custom DNS Names with Dedicated SSL Certificates: Use custom DNS names and dedicated SSL certificates to serve your content securely over HTTPS.
- Restricting Content Access with Signed URLs and Signed Cookies: Restrict access to your content by using signed URLs and signed cookies.
- Origin Access Identity (OAI): A special CloudFront user that you can associate with your distributions to restrict access to content in Amazon S3 buckets.
- High Availability with CloudFront Origin Failover: Improve the availability of your application by configuring CloudFront to use origin failover.
