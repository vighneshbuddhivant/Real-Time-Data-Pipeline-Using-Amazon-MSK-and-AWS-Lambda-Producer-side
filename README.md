# Real-Time Data Pipeline Using Amazon MSK and AWS Lambda Consumer Side

## Problem Overview
In modern data-driven applications, processing data in real-time is crucial for timely insights and decision-making. This project demonstrates a real-time data pipeline using Amazon Managed Streaming for Apache Kafka (MSK) and AWS Lambda. The architecture captures data from a third-party application (via an API or through SQS/DynamoDB), processes it, and sends it to an MSK topic for further analysis or storage in a data warehouse.

## Architecture
![](https://github.com/vighneshbuddhivant/Real-Time-Data-Pipeline-Using-Amazon-MSK-and-AWS-Lambda-Producer-side/blob/64fc95beb74751758ef381ca2d9dbe92651d6bfc/lambda-msk-arch.png)

1. **Data Source**: A third-party application or web browser sends data through an API to an AWS service (such as SQS or DynamoDB).
2. **Lambda Function**: An AWS Lambda function captures the incoming requests, processes the data, and publishes it to an MSK topic.
3. **MSK Cluster**: The data is stored in an Amazon MSK topic for further processing.
4. **Data Consumption**: Consumers (e.g., another Lambda function, EC2 instance, or data warehouse) can read from the MSK topic for further analysis or storage.

## VPC and Networking Configuration
1. **Create a VPC**:
   - **Name**: `virtual-private-cloud-lambda`
   - **CIDR**: `11.0.0.0/16`
  
2. **Create Subnets**:
   - **Public Subnet A**: `11.0.0.0/24`
   - **Public Subnet B**: `11.0.1.0/24`
   - **Private Subnet A**: `11.0.2.0/24`
   - **Private Subnet B**: `11.0.3.0/24`

3. **Configure Internet Gateway**: Attach an internet gateway to the VPC and associate it with the public subnets.

## MSK Cluster Creation
1. Create an Amazon MSK cluster following the required specifications for the consumer-side project.
![](https://github.com/vighneshbuddhivant/Real-Time-Data-Pipeline-Using-Amazon-MSK-and-AWS-Lambda-Producer-side/blob/0be75c4f722755bbda3571e4e0f2b775a8606a6a/msk-cluster.png)
  
## EC2 Instance Configuration
1. Launch an EC2 instance within the same VPC to act as a client machine.
2. Edit the security group to allow inbound traffic from the MSK cluster.
3. Configure the MSK security group to allow inbound traffic from the EC2 instance.

## Kafka Setup on EC2
1. **SSH into the EC2 Instance**:
   - Use PuTTY to connect to the instance using its IPv4 address and your `.ppk` file.

2. **Install Java**:
   ```bash
   sudo amazon-linux-extras enable corretto8
   sudo yum install -y java-1.8.0-amazon-corretto-devel
   ```

3. **Download and Extract Kafka**:
   ```bash
   wget https://archive.apache.org/dist/kafka/3.5.1/kafka_2.12-3.5.1.tgz
   tar -xzf kafka_2.12-3.5.1.tgz
   cd kafka_2.12-3.5.1
   ```

4. **Create Kafka Topic**:
   ```bash
   bin/kafka-topics.sh --create --topic demo_testing1 --bootstrap-server b-1.mskprojectproducerclu.b88034.c3.kafka.ap-south-1.amazonaws.com:9092,b-2.mskprojectproducerclu.b88034.c3.kafka.ap-south-1.amazonaws.com:9092 --replication-factor 1 --partitions 1
   ```

5. **Publish Messages to Kafka Topic**:
   ```python
   from time import sleep
   from json import dumps
   from kafka import KafkaProducer

   topic_name = 'demo_testing1'
   producer = KafkaProducer(bootstrap_servers=['b-1.mskprojectproducerclu.b88034.c3.kafka.ap-south-1.amazonaws.com:9092','b-2.mskprojectproducerclu.b88034.c3.kafka.ap-south-1.amazonaws.com:9092'], value_serializer=lambda x: dumps(x).encode('utf-8'))

   for e in range(1000):
       data = {'number': e}
       print(data)
       producer.send(topic_name, value=data)
       sleep(1)
   ```

6. **Consume Messages from Kafka Topic**:
   ```bash
   bin/kafka-console-consumer.sh --topic demo_testing1 --bootstrap-server b-1.mskprojectproducerclu.b88034.c3.kafka.ap-south-1.amazonaws.com:9092,b-2.mskprojectproducerclu.b88034.c3.kafka.ap-south-1.amazonaws.com:9092
   ```

## Lambda Layer for Kafka
1. To run the producer code in AWS Lambda, create a Lambda layer for the `kafka-python` module:
   - Use WSL to create a virtual environment and install the module.
   
### WSL Commands
```bash
# Start WSL
wsl

# Update package lists
sudo apt-get update

# Install virtualenv
sudo apt install python3-virtualenv

# Create a virtual environment
virtualenv kafka_yt

# Activate the virtual environment
source kafka_yt/bin/activate

# Install pip (if not already installed)
sudo apt install python3-pip

# Upgrade pip
python3 -m pip install --upgrade pip

# Create directory for Lambda layer
mkdir -p lambda_layers/python/lib/python3.8/site-packages

# Navigate to the site-packages directory
cd lambda_layers/python/lib/python3.8/site-packages

# Install kafka-python to the site-packages directory
pip install kafka-python -t .

# Install zip (if not already installed)
sudo apt install zip
```

2. After creating the Lambda layer, upload the zip file to an S3 bucket.

## Lambda Function Configuration
1. Create a Lambda function and add the layer from the S3 bucket.
2. Provide full VPC access to the Lambda function role.
3. Add the VPC and subnets created earlier to the Lambda function, and configure the security groups to allow traffic to and from the MSK cluster.
4. Test the Lambda function manually to ensure data is being published to the MSK topic.

## Conclusion
This project illustrates the integration of AWS Lambda with Amazon MSK to create a real-time data processing pipeline. The architecture provides a scalable solution for capturing and processing data from various sources, demonstrating the capabilities of serverless computing and managed streaming. This setup can serve as a foundation for more complex system designs in data engineering.
