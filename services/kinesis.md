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

# Amazon Kinesis Overview

Amazon Kinesis is a powerful service for real-time data streaming and processing. This document provides an overview of the capacities and limitations for the various components of Amazon Kinesis.

## Kinesis Data Streams

### Capacity
1. **Shards**:
   - Each shard can support up to 1 MB/sec or 1,000 records/sec for writes.
   - Each shard can support up to 2 MB/sec for reads.

2. **Records**:
   - Maximum record size is 1 MB.
   - Each record can include up to 1 MB of data, including partition keys and sequence numbers.

3. **Retention Period**:
   - Default retention period is 24 hours.
   - Can be extended up to 7 days.
   - With extended retention, data can be retained for up to 365 days.

4. **Provisioned Throughput**:
   - You can increase or decrease the number of shards in your stream to scale the throughput.

### Limitations
1. **Shard Limits**:
   - There is a soft limit on the number of shards per stream (default is 500 shards per AWS account per region). This limit can be increased by contacting AWS support.

2. **Data Retention**:
   - Data older than the retention period is automatically deleted.

3. **Latency**:
   - There is a slight delay (typically less than 1 second) from the time data is written to a shard until it is available for reading.

4. **Cost**:
   - Costs are based on the number of shards, PUT payload units, and data retrievals.

## Kinesis Data Firehose

### Capacity
1. **Data Delivery**:
   - Supports data delivery to Amazon S3, Amazon Redshift, Amazon Elasticsearch Service, and Splunk.
   - Can handle gigabytes of data per second.

2. **Data Transformation**:
   - Supports data transformation using AWS Lambda functions.

3. **Buffering**:
   - Configurable buffering in terms of size (1 MB to 128 MB) and time (60 seconds to 900 seconds).

### Limitations
1. **Record Size**:
   - Maximum record size is 1 MB.

2. **Data Transformation**:
   - Lambda function for data transformation has a timeout limit of 5 minutes.

3. **Destination Limits**:
   - Each delivery stream can have only one destination.

4. **Cost**:
   - Costs are based on the amount of data ingested, data format conversion, and data delivery.

## Kinesis Data Analytics

### Capacity
1. **Real-Time Processing**:
   - Can process streaming data in real-time using SQL queries.

2. **Input and Output Streams**:
   - Supports Kinesis Data Streams and Kinesis Data Firehose as input and output.

3. **Parallelism**:
   - Can scale the number of parallel tasks to process data.

### Limitations
1. **SQL Query Complexity**:
   - Complex SQL queries can impact performance and may require optimization.

2. **Resource Limits**:
   - There are limits on the number of applications, parallel tasks, and input/output streams per AWS account per region.

3. **Latency**:
   - There is a slight delay in processing due to the nature of real-time analytics.

4. **Cost**:
   - Costs are based on the amount of data processed and the resources used.

## Kinesis Video Streams

### Capacity
1. **Data Ingestion**:
   - Can ingest video streams from millions of devices.

2. **Storage**:
   - Supports long-term storage of video data in Amazon S3.

3. **Playback**:
   - Supports real-time playback and retrieval of video data.

### Limitations
1. **Latency**:
   - There can be a slight delay in video streaming and playback.

2. **Data Retention**:
   - Data retention period is configurable, but there are limits on the maximum retention period.

3. **Cost**:
   - Costs are based on the amount of data ingested, stored, and retrieved.

## General Limitations

1. **Service Limits**:
   - Each AWS account has default limits on the number of Kinesis resources (streams, shards, delivery streams, etc.) that can be created. These limits can be increased by contacting AWS support.

2. **Data Transfer**:
   - Data transfer between AWS regions incurs additional costs.

3. **Security**:
   - Proper IAM policies and encryption should be used to secure data in transit and at rest.


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

# Amazon Kinesis Pricing Models

Amazon Kinesis offers different pricing models based on the specific service you are using. Below are the pricing models for the various components of Amazon Kinesis.

## Kinesis Data Streams

### Pricing Components
1. **Shard Hour**:
   - You are charged for each shard hour that your stream is active. The cost is based on the number of shards and the duration they are active.

2. **PUT Payload Units**:
   - You are charged for the number of PUT payload units. A PUT payload unit is 25 KB of data. If your record size exceeds 25 KB, you will be charged for additional units.

3. **Extended Data Retention**:
   - If you extend the data retention period beyond the default 24 hours, you will be charged for each shard hour of extended retention.

4. **Enhanced Fan-Out**:
   - You are charged for each consumer-shard hour when using enhanced fan-out, which allows multiple consumers to read data from the stream with dedicated throughput.

5. **Retrieval of Data**:
   - You are charged for the amount of data retrieved from the stream. This includes both standard and enhanced fan-out consumers.

### Example Pricing
- **Shard Hour**: $0.015 per shard hour.
- **PUT Payload Unit**: $0.014 per million PUT payload units.
- **Extended Data Retention**: $0.021 per shard hour for extended retention.
- **Enhanced Fan-Out**: $0.015 per consumer-shard hour.
- **Retrieval of Data**: $0.013 per GB of data retrieved.

## Kinesis Data Firehose

### Pricing Components
1. **Ingested Data**:
   - You are charged based on the volume of data ingested into the delivery stream.

2. **Data Format Conversion**:
   - If you enable data format conversion (e.g., converting JSON to Parquet or ORC), you are charged based on the volume of data processed.

3. **Data Delivery**:
   - You are charged based on the volume of data delivered to the destination (e.g., S3, Redshift, Elasticsearch, Splunk).

### Example Pricing
- **Ingested Data**: $0.029 per GB of data ingested.
- **Data Format Conversion**: $0.018 per GB of data processed.
- **Data Delivery**: $0.029 per GB of data delivered.

## Kinesis Data Analytics

### Pricing Components
1. **Running Application**:
   - You are charged based on the number of Kinesis Processing Units (KPUs) used by your application. A KPU consists of 1 vCPU and 4 GB of memory.

2. **Input and Output Data**:
   - You are charged based on the volume of data processed by your application.

### Example Pricing
- **Running Application**: $0.11 per KPU hour.
- **Input and Output Data**: $0.011 per GB of data processed.

## Kinesis Video Streams

### Pricing Components
1. **Data Ingestion**:
   - You are charged based on the volume of video data ingested into the stream.

2. **Data Storage**:
   - You are charged based on the volume of video data stored in the stream.

3. **Data Retrieval**:
   - You are charged based on the volume of video data retrieved from the stream.

4. **Playback**:
   - You are charged based on the volume of video data played back.

### Example Pricing
- **Data Ingestion**: $0.0085 per GB of data ingested.
- **Data Storage**: $0.023 per GB per month.
- **Data Retrieval**: $0.011 per GB of data retrieved.
- **Playback**: $0.011 per GB of data played back.

## Additional Costs

1. **Data Transfer**:
   - Data transfer between AWS regions incurs additional costs.
   - Data transfer within the same region is typically free.

2. **AWS Lambda**:
   - If you use AWS Lambda for data transformation in Kinesis Data Firehose or Kinesis Data Streams, you will incur additional charges based on Lambda's pricing model.

## Summary

- **Kinesis Data Streams**: Charged based on shard hours, PUT payload units, extended data retention, enhanced fan-out, and data retrieval.
- **Kinesis Data Firehose**: Charged based on ingested data, data format conversion, and data delivery.
- **Kinesis Data Analytics**: Charged based on running application (KPU hours) and input/output data processed.
- **Kinesis Video Streams**: Charged based on data ingestion, data storage, data retrieval, and playback.

