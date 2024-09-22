# Amazon Kinesis

Amazon Kinesis is a platform on AWS to collect, process, and analyze real-time, streaming data. It enables you to build real-time applications that can continuously ingest and process large streams of data records in real-time.

## Features

- **Real-Time Processing**: Process and analyze streaming data in real-time.
- **Scalability**: Seamlessly scales to handle any amount of streaming data.
- **Durability**: Data is replicated across multiple availability zones for durability.
- **Integration**: Easily integrates with other AWS services like Lambda, S3, Redshift, and more.
- **Cost-Effective**: Pay only for the resources you use.

## Components

1. **Kinesis Data Streams**: Enables you to build custom, real-time applications that process or analyze streaming data for specialized needs.
2. **Kinesis Data Firehose**: The easiest way to reliably load streaming data into data lakes, data stores, and analytics services.
3. **Kinesis Data Analytics**: Allows you to process and analyze streaming data using standard SQL.
4. **Kinesis Video Streams**: Makes it easy to securely stream video from connected devices to AWS for analytics, machine learning (ML), and other processing.

## Creating a Kinesis Data Stream Using AWS CDK (Go)

### Prerequisites

- AWS CDK installed. If not, install it using `npm install -g aws-cdk`.
- Go programming language installed.

### Steps

1. **Set Up Your CDK Project**:
   - Create a new CDK project using `cdk init app --language go`.

2. **Install Required CDK Libraries**:
   - Install the necessary CDK libraries for Amazon Kinesis.
     ```sh
     go get github.com/aws/aws-cdk-go/awscdk/v2
     go get github.com/aws/constructs-go/constructs/v10
     ```

3. **Define the Kinesis Stream in Your CDK Stack**:
   - Open the `main.go` file and define your stack. Below is an example of how to create a Kinesis stream using AWS CDK in Go.

### Example CDK Stack (Go)

```go
package main

import (
    "github.com/aws/aws-cdk-go/awscdk/v2"
    "github.com/aws/aws-cdk-go/awscdk/v2/awskinesis"
    "github.com/aws/constructs-go/constructs/v10"
)

type KinesisStreamStackProps struct {
    awscdk.StackProps
}

func NewKinesisStreamStack(scope constructs.Construct, id string, props *KinesisStreamStackProps) awscdk.Stack {
    stack := awscdk.NewStack(scope, &id, &props.StackProps)

    // Create a Kinesis stream
    awskinesis.NewStream(stack, jsii.String("MyKinesisStream"), &awskinesis.StreamProps{
        StreamName: jsii.String("my-kinesis-stream"),
        ShardCount: jsii.Number(1),
    })

    return stack
}

func main() {
    app := awscdk.NewApp(nil)

    NewKinesisStreamStack(app, "KinesisStreamStack", &KinesisStreamStackProps{
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


client script

```python

import boto3
import json
import time

# Initialize the Kinesis client
kinesis_client = boto3.client('kinesis', region_name='us-west-2')

# Stream name
stream_name = 'my-kinesis-stream'

# Function to put records into the stream
def put_records():
    for i in range(10):
        data = {
            'id': i,
            'message': f'This is record {i}'
        }
        kinesis_client.put_record(
            StreamName=stream_name,
            Data=json.dumps(data),
            PartitionKey=str(i)
        )
        print(f'Put record {i}')
        time.sleep(1)

if __name__ == '__main__':
    put_records()


```
