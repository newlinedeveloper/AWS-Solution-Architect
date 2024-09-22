# AWS Glue

AWS Glue is a fully managed extract, transform, and load (ETL) service that makes it easy to prepare and load your data for analytics. It simplifies the process of building and managing ETL jobs by providing a serverless environment that automatically scales to handle your data processing needs.

## Features

- **Serverless**: No infrastructure to manage; AWS Glue automatically provisions and scales the resources needed for your ETL jobs.
- **Integrated Data Catalog**: Automatically discovers and catalogs metadata about your data stores.
- **Job Scheduling**: Allows you to schedule and manage ETL jobs.
- **Developer Endpoints**: Provides endpoints for interactive development and testing.
- **Built-in Transformations**: Offers a variety of built-in transformations to clean and enrich your data.

## Creating an AWS Glue Job Using AWS CDK (Go)

### Prerequisites

- AWS CDK installed. If not, install it using `npm install -g aws-cdk`.
- Go programming language installed.

### Steps

1. **Set Up Your CDK Project**:
   - Create a new CDK project using `cdk init app --language go`.

2. **Install Required CDK Libraries**:
   - Install the necessary CDK libraries for AWS Glue and other services.
     ```sh
     go get github.com/aws/aws-cdk-go/awscdk/v2
     go get github.com/aws/constructs-go/constructs/v10
     ```

3. **Define the Glue Job in Your CDK Stack**:
   - Open the `main.go` file and define your stack. Below is an example of how to create a Glue job using AWS CDK in Go.

### Example CDK Stack (Go)

```go
package main

import (
    "fmt"

    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsglue"
    "github.com/aws/aws-cdk-go/awscdk/v2/awsiam"
    "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
    "github.com/aws/constructs-go/constructs/v10"
)

type GlueJobStackProps struct {
    awscdk.StackProps
}

func NewGlueJobStack(scope constructs.Construct, id string, props *GlueJobStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create an S3 bucket for Glue job scripts and data
    scriptBucket := awss3.NewBucket(stack, jsii.String("GlueScriptBucket"), &awss3.BucketProps{})

    // Create an IAM role for the Glue job
    glueRole := awsiam.NewRole(stack, jsii.String("GlueJobRole"), &awsiam.RoleProps{
        AssumedBy: awsiam.NewServicePrincipal(jsii.String("glue.amazonaws.com"), nil),
        ManagedPolicies: &[]awsiam.IManagedPolicy{
            awsiam.ManagedPolicy_FromAwsManagedPolicyName(jsii.String("service-role/AWSGlueServiceRole")),
        },
    })

    // Grant the role read/write permissions to the S3 bucket
    scriptBucket.GrantReadWrite(glueRole)

    // Define the Glue job
    awsglue.NewCfnJob(stack, jsii.String("MyGlueJob"), &awsglue.CfnJobProps{
        Name: jsii.String("my-glue-job"),
        Role: glueRole.RoleArn(),
        Command: &awsglue.CfnJob_JobCommandProperty{
            Name:           jsii.String("glueetl"),
            ScriptLocation: jsii.String(fmt.Sprintf("s3://%s/scripts/my_glue_script.py", *scriptBucket.BucketName())),
        },
        DefaultArguments: map[string]interface{}{
            "--job-language":    "python",
            "--extra-py-files":  fmt.Sprintf("s3://%s/dependencies.zip", *scriptBucket.BucketName()),
        },
        MaxRetries:   jsii.Number(1),
        MaxCapacity:  jsii.Number(10.0),
        GlueVersion:  jsii.String("2.0"),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewGlueJobStack(app, "GlueJobStack", &GlueJobStackProps{
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


Glue script

```
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

# Read data from S3
datasource0 = glueContext.create_dynamic_frame.from_catalog(database = "my_database", table_name = "my_table", transformation_ctx = "datasource0")

# Apply transformations
applymapping1 = ApplyMapping.apply(frame = datasource0, mappings = [("col0", "string", "col0", "string"), ("col1", "int", "col1", "int")], transformation_ctx = "applymapping1")

# Write data to S3
datasink2 = glueContext.write_dynamic_frame.from_options(frame = applymapping1, connection_type = "s3", connection_options = {"path": "s3://my-target-bucket/transformed/"}, format = "json", transformation_ctx = "datasink2")

job.commit()

```

upload script to s3 bucket

```
aws s3 cp my_glue_script.py s3://<your-script-bucket-name>/scripts/my_glue_script.py
```

