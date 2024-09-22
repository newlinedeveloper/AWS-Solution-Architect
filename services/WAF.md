# AWS Web Application Firewall (WAF)

AWS Web Application Firewall (WAF) is a web application firewall that helps protect your web applications or APIs against common web exploits and vulnerabilities. AWS WAF gives you control over which traffic to allow or block to your web applications by defining customizable web security rules.

## Features

- **Protection Against Common Web Exploits**: Protects against SQL injection, cross-site scripting (XSS), and other common web exploits.
- **Customizable Rules**: Allows you to create custom rules to filter web traffic based on specific conditions.
- **Managed Rules**: Provides pre-configured rules managed by AWS and AWS Marketplace sellers.
- **Real-Time Visibility**: Offers real-time visibility into web traffic and security metrics.
- **Integration with AWS Services**: Integrates with Amazon CloudFront, Application Load Balancer (ALB), API Gateway, and AWS AppSync.

## AWS WAF Rule Statements To Filter Web Traffic

AWS WAF rule statements allow you to define conditions to filter web traffic. These rule statements can be combined to create complex filtering rules.

### Common Rule Statements

1. **ByteMatchStatement**
   - Filters web requests based on the presence of a string or pattern in a specific part of the request.

2. **GeoMatchStatement**
   - Filters web requests based on the geographic location of the request origin.

3. **IPSetReferenceStatement**
   - Filters web requests based on the IP addresses or IP address ranges specified in an IP set.

4. **RegexPatternSetReferenceStatement**
   - Filters web requests based on patterns specified in a regex pattern set.

5. **SizeConstraintStatement**
   - Filters web requests based on the size of a specific part of the request.

6. **SQLiMatchStatement**
   - Filters web requests to detect SQL injection attacks.

7. **XssMatchStatement**
   - Filters web requests to detect cross-site scripting (XSS) attacks.

8. **RateBasedStatement**
   - Filters web requests based on the rate of requests from a single IP address.

### Example: Creating an AWS WAF Web ACL with Rule Statements Using AWS CDK (Go)

The following example demonstrates how to create an AWS WAF Web ACL with various rule statements using AWS CDK in Go.

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awswafv2"
    "github.com/aws/constructs-go/constructs/v10"
)

type WafStackProps struct {
    awscdk.StackProps
}

func NewWafStack(scope constructs.Construct, id string, props *WafStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an IP set
    ipSet := awswafv2.NewCfnIPSet(stack, jsii.String("MyIPSet"), &awswafv2.CfnIPSetProps{
        Addresses: &[]*string{
            jsii.String("192.0.2.0/24"),
        },
        IPVersion: jsii.String("IPV4"),
        Scope:     jsii.String("REGIONAL"),
        Name:      jsii.String("MyIPSet"),
    })

    // Create a Web ACL
    webAcl := awswafv2.NewCfnWebACL(stack, jsii.String("MyWebACL"), &awswafv2.CfnWebACLProps{
        DefaultAction: &awswafv2.CfnWebACL_DefaultActionProperty{
            Allow: &awswafv2.CfnWebACL_AllowActionProperty{},
        },
        Scope: jsii.String("REGIONAL"),
        Rules: &[]*awswafv2.CfnWebACL_RuleProperty{
            {
                Name:     jsii.String("IPSetRule"),
                Priority: jsii.Number(1),
                Action: &awswafv2.CfnWebACL_RuleActionProperty{
                    Block: &awswafv2.CfnWebACL_BlockActionProperty{},
                },
                Statement: &awswafv2.CfnWebACL_StatementProperty{
                    IPSetReferenceStatement: &awswafv2.CfnWebACL_IPSetReferenceStatementProperty{
                        ARN: ipSet.AttrArn(),
                    },
                },
                VisibilityConfig: &awswafv2.CfnWebACL_VisibilityConfigProperty{
                    SampledRequestsEnabled: jsii.Bool(true),
                    CloudWatchMetricsEnabled: jsii.Bool(true),
                    MetricName: jsii.String("IPSetRule"),
                },
            },
            {
                Name:     jsii.String("SQLiMatchRule"),
                Priority: jsii.Number(2),
                Action: &awswafv2.CfnWebACL_RuleActionProperty{
                    Block: &awswafv2.CfnWebACL_BlockActionProperty{},
                },
                Statement: &awswafv2.CfnWebACL_StatementProperty{
                    SqliMatchStatement: &awswafv2.CfnWebACL_SqliMatchStatementProperty{
                        FieldToMatch: &awswafv2.CfnWebACL_FieldToMatchProperty{
                            UriPath: &awswafv2.CfnWebACL_UriPathProperty{},
                        },
                        TextTransformations: &[]*awswafv2.CfnWebACL_TextTransformationProperty{
                            {
                                Priority: jsii.Number(0),
                                Type:     jsii.String("URL_DECODE"),
                            },
                        },
                    },
                },
                VisibilityConfig: &awswafv2.CfnWebACL_VisibilityConfigProperty{
                    SampledRequestsEnabled: jsii.Bool(true),
                    CloudWatchMetricsEnabled: jsii.Bool(true),
                    MetricName: jsii.String("SQLiMatchRule"),
                },
            },
        },
        VisibilityConfig: &awswafv2.CfnWebACL_VisibilityConfigProperty{
            SampledRequestsEnabled: jsii.Bool(true),
            CloudWatchMetricsEnabled: jsii.Bool(true),
            MetricName: jsii.String("MyWebACL"),
        },
        Name: jsii.String("MyWebACL"),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewWafStack(app, "WafStack", &WafStackProps{
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

- AWS WAF: A web application firewall that helps protect your web applications or APIs against common web exploits and vulnerabilities.
- Rule Statements: Define conditions to filter web traffic, including ByteMatchStatement, GeoMatchStatement, IPSetReferenceStatement, RegexPatternSetReferenceStatement, SizeConstraintStatement, SQLiMatchStatement, XssMatchStatement, and RateBasedStatement.
