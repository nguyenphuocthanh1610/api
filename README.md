
const { DateTime } = require('luxon');

exports.handler = async (event) => {
    // Chuỗi ngày giờ đầu vào
    const inputDateTimeString = '2024-07-31 12:34:56'; // Chuỗi ngày giờ bất kỳ
    const inputTimeZone = 'Asia/Ho_Chi_Minh'; // Múi giờ của chuỗi ngày giờ đầu vào

    // Chuyển đổi chuỗi ngày giờ sang đối tượng DateTime trong múi giờ của chuỗi
    const dateTime = DateTime.fromFormat(inputDateTimeString, 'yyyy-MM-dd HH:mm:ss', { zone: inputTimeZone });

    // Chuyển đổi ngày giờ sang UTC và định dạng lại
    const dateTimeInUTC = dateTime.toUTC().toISO();

    console.log("Chuỗi ngày giờ đầu vào:", inputDateTimeString);
    console.log("Múi giờ đầu vào:", inputTimeZone);
    console.log("Ngày giờ chuyển đổi sang UTC:", dateTimeInUTC);

    return {
        statusCode: 200,
        body: JSON.stringify({
            message: "Ngày giờ đã được chuyển đổi sang UTC",
            dateTimeInUTC: dateTimeInUTC,
        }),
    };
};

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt"
      ],
      "Resource": "arn:aws:kms:us-west-2:123456789012:key/abcd1234-5678-90ab-cdef-1234567890ab"
    }
  ]
}


// Import các thư viện cần thiết từ AWS SDK
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
const { fromIni } = require('@aws-sdk/credential-providers');
const fs = require('fs');

// Cấu hình client S3
const s3Client = new S3Client({
  region: 'your-region', // Ví dụ: 'us-east-1'
  credentials: fromIni({ profile: 'your-profile' }), // Đọc từ file config (~/.aws/credentials)
});

// Định nghĩa thông tin cho object cần upload
const bucketName = 'your-bucket-name';
const keyName = 'your-object-key';
const filePath = 'path/to/your/file.ext';

// Đọc file từ hệ thống
const fileStream = fs.createReadStream(filePath);

// Định nghĩa lệnh PutObjectCommand
const uploadParams = {
  Bucket: bucketName,
  Key: keyName,
  Body: fileStream,
  ContentType: 'your-content-type', // Ví dụ: 'image/jpeg'
};

// Hàm upload object lên S3
const run = async () => {
  try {
    const data = await s3Client.send(new PutObjectCommand(uploadParams));
    console.log('Upload thành công', data);
  } catch (err) {
    console.error('Upload thất bại', err);
  }
};

// Chạy hàm upload
run();

import boto3
import paramiko
import os

def lambda_handler(event, context):
    # Thông tin về bucket và key của file CSV trong S3
    bucket_name = event['bucket']
    object_key = event['key']
    local_file_path = '/tmp/' + os.path.basename(object_key)

    # Tải file từ S3
    s3 = boto3.client('s3')
    s3.download_file(bucket_name, object_key, local_file_path)

    # Cấu hình kết nối SSH
    hostname = "your.ec2.public.ip"
    port = 22
    username = "your_ec2_username"
    key_path = "/path/to/your/private/key.pem"
    remote_file_path = "/remote/path/to/" + os.path.basename(object_key)

    # Kết nối và tải lên tệp
    try:
        key = paramiko.RSAKey.from_private_key_file(key_path)
        transport = paramiko.Transport((hostname, port))
        transport.connect(username=username, pkey=key)
        sftp = paramiko.SFTPClient.from_transport(transport)
        sftp.put(local_file_path, remote_file_path)
        sftp.close()
        transport.close()
        return {"status": "success"}
    except Exception as e:
        print(f"Error: {e}")
        return {"status": "error", "error": str(e)}

import boto3
import paramiko
import os

def lambda_handler(event, context):
    # Thông tin về bucket và key của file CSV trong S3
    bucket_name = event['bucket']
    object_key = event['key']
    local_file_path = '/tmp/' + os.path.basename(object_key)

    # Tải file từ S3
    s3 = boto3.client('s3')
    s3.download_file(bucket_name, object_key, local_file_path)

    # Cấu hình kết nối SFTP
    hostname = "your.server.com"
    port = 22
    username = "your_username"
    password = "your_password"
    remote_file_path = "/remote/path/to/" + os.path.basename(object_key)

    # Kết nối và tải lên tệp
    try:
        transport = paramiko.Transport((hostname, port))
        transport.connect(username=username, password=password)
        sftp = paramiko.SFTPClient.from_transport(transport)
        sftp.put(local_file_path, remote_file_path)
        sftp.close()
        transport.close()
        return {"status": "success"}
    except Exception as e:
        print(f"Error: {e}")
        return {"status": "error", "error": str(e)}


import paramiko
import os

def lambda_handler(event, context):
    # Đường dẫn tệp CSV cần tải lên
    local_file_path = "/tmp/my_file.csv"

    # Đọc hoặc tạo tệp CSV ở đây
    with open(local_file_path, 'w') as file:
        file.write("col1,col2,col3\nval1,val2,val3\n")

    # Cấu hình kết nối SFTP
    hostname = "your.server.com"
    port = 22
    username = "your_username"
    password = "your_password"
    remote_file_path = "/remote/path/to/my_file.csv"

    # Kết nối và tải lên tệp
    try:
        transport = paramiko.Transport((hostname, port))
        transport.connect(username=username, password=password)
        sftp = paramiko.SFTPClient.from_transport(transport)
        sftp.put(local_file_path, remote_file_path)
        sftp.close()
        transport.close()
        return {"status": "success"}
    except Exception as e:
        print(f"Error: {e}")
        return {"status": "error", "error": str(e)}


AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Resources:
  MomentLayer:
    Type: 'AWS::Serverless::LayerVersion'
    Properties:
      LayerName: moment-layer
      Description: A Lambda layer containing the moment.js library
      ContentUri: layer/
      CompatibleRuntimes:
        - nodejs14.x
        - nodejs18.x
        - nodejs20.x
      LicenseInfo: 'MIT'

  LambdaFunction:
    Type: 'AWS::Serverless::Function'
    Properties:
      Handler: index.handler
      Runtime: nodejs18.x
      CodeUri: lambda/
      Layers:
        - !Ref MomentLayer
      Role: arn:aws:iam::123456789012:role/lambda-role
      Description: Lambda function using moment.js from a layer
      MemorySize: 128
      Timeout: 100

sam package \
    --template-file template.yaml \
    --output-template-file packaged.yaml \
    --s3-bucket <YOUR_S3_BUCKET_NAME>
sam deploy \
    --template-file packaged.yaml \
    --stack-name my-sam-app \
    --capabilities CAPABILITY_IAM
    
Resources:
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonS3FullAccess
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
        - arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole

import boto3
from boto3.session import Session

# Thông tin role cần assume
role_arn = 'arn:aws:iam::123456789012:role/YourRoleName'
role_session_name = 'YourSessionName'

# Tạo client cho STS (Security Token Service)
sts_client = boto3.client('sts')

# Assume role
assumed_role_object = sts_client.assume_role(
    RoleArn=role_arn,
    RoleSessionName=role_session_name
)

# Lấy các thông tin về temporary credentials
credentials = assumed_role_object['Credentials']

# Tạo một session mới với các temporary credentials
session = Session(
    aws_access_key_id=credentials['AccessKeyId'],
    aws_secret_access_key=credentials['SecretAccessKey'],
    aws_session_token=credentials['SessionToken']
)

# Tạo S3 client với session mới
s3_client = session.client('s3')

# Đường dẫn file bạn muốn upload
file_path = 'path/to/your/file.txt'
# Tên bucket và key (tên file trong S3)
bucket_name = 'your-bucket-name'
s3_key = 'your-file-name.txt'

# Upload file lên S3
s3_client.upload_file(file_path, bucket_name, s3_key)

print(f"File {file_path} đã được upload lên {bucket_name}/{s3_key}")

import pandas as pd

# Tạo DataFrame từ dữ liệu cho trước
data = {
    'a': [1, 4, 7],
    'b': [2, 5, 8],
    'c': [3, 6, 9]
}
index = [
    '2024-01-01T00:00:00',
    '2024-01-01T00:30:00',
    '2024-01-01T01:00:00'
]
df = pd.DataFrame(data, index=index)

# Chuyển đổi cột thời gian thành các cột riêng biệt
df_transposed = df.T
df_transposed.columns = ['0:00', '00:30', '01:00']

# Xuất ra file CSV
df_transposed.to_csv('output.csv', index=False)

print(df_transposed)


import pandas as pd

# Tạo một DataFrame mẫu
data = {
    'A': [1, 2, 3],
    'B': [4, 5, 6],
    'C': [7, 8, 9]
}

df = pd.DataFrame(data)
print("DataFrame ban đầu:")
print(df)
# Xoay DataFrame
df_transposed = df.transpose()
print("DataFrame sau khi xoay:")
print(df_transposed)

df_transposed = df.T
print("DataFrame sau khi xoay:")
print(df_transposed)



AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation template to configure VPC for Lambda in different environments with two subnets.

Parameters:
  Environment:
    Description: The environment to deploy (local, staging, production)
    Type: String
    AllowedValues:
      - local
      - staging
      - production
    Default: local

  VpcIdLocal:
    Description: VPC ID for local environment
    Type: String
    Default: vpc-xxxxxxxxxx

  SubnetIdLocal1:
    Description: First Subnet ID for local environment
    Type: String
    Default: subnet-xxxxxxxxxx

  SubnetIdLocal2:
    Description: Second Subnet ID for local environment
    Type: String
    Default: subnet-yyyyyyyyyy

  SecurityGroupIdLocal:
    Description: Security Group ID for local environment
    Type: String
    Default: sg-xxxxxxxxxx

  VpcIdStaging:
    Description: VPC ID for staging environment
    Type: String
    Default: vpc-yyyyyyyyyy

  SubnetIdStaging1:
    Description: First Subnet ID for staging environment
    Type: String
    Default: subnet-zzzzzzzzzz

  SubnetIdStaging2:
    Description: Second Subnet ID for staging environment
    Type: String
    Default: subnet-aaaaaaaaaa

  SecurityGroupIdStaging:
    Description: Security Group ID for staging environment
    Type: String
    Default: sg-yyyyyyyyyy

  VpcIdProduction:
    Description: VPC ID for production environment
    Type: String
    Default: vpc-zzzzzzzzzz

  SubnetIdProduction1:
    Description: First Subnet ID for production environment
    Type: String
    Default: subnet-bbbbbbbbbb

  SubnetIdProduction2:
    Description: Second Subnet ID for production environment
    Type: String
    Default: subnet-cccccccccc

  SecurityGroupIdProduction:
    Description: Security Group ID for production environment
    Type: String
    Default: sg-zzzzzzzzzz

Conditions:
  IsLocal: !Equals [!Ref Environment, local]
  IsStaging: !Equals [!Ref Environment, staging]
  IsProduction: !Equals [!Ref Environment, production]

Resources:
  MyLambdaFunction:
    Type: AWS::Lambda::Function
    Properties: 
      FunctionName: MyFunction
      Handler: index.handler
      Role: arn:aws:iam::account-id:role/lambda-execution-role
      Code:
        S3Bucket: my-lambda-functions
        S3Key: function.zip
      Runtime: nodejs14.x
      VpcConfig:
        SecurityGroupIds:
          - !If 
            - IsLocal
            - !Ref SecurityGroupIdLocal
            - !If 
              - IsStaging
              - !Ref SecurityGroupIdStaging
              - !Ref SecurityGroupIdProduction
        SubnetIds:
          - !If 
            - IsLocal
            - !Ref SubnetIdLocal1
            - !If 
              - IsStaging
              - !Ref SubnetIdStaging1
              - !Ref SubnetIdProduction1
          - !If 
            - IsLocal
            - !Ref SubnetIdLocal2
            - !If 
              - IsStaging
              - !Ref SubnetIdStaging2
              - !Ref SubnetIdProduction2

Outputs:
  LambdaFunctionName:
    Description: The name of the Lambda function
    Value: !Ref MyLambdaFunction

AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyLambdaFunction:
    Type: 'AWS::Lambda::Function'
    Properties:
      FunctionName: MyLambdaFunction
      Handler: index.handler
      Role: arn:aws:iam::123456789012:role/service-role/MyLambdaRole
      Runtime: python3.8
      Code:
        S3Bucket: my-bucket
        S3Key: my-lambda-code.zip

  MyCloudWatchEventRule:
    Type: 'AWS::Events::Rule'
    Properties:
      Description: Schedule rule for triggering Lambda Function daily
      ScheduleExpression: cron(0 7 * * ? *)  # Example: Trigger daily at 7:00 AM UTC
      State: ENABLED
      Targets:
        - Arn: !GetAtt MyLambdaFunction.Arn
          Id: MyLambdaFunctionTarget

  LambdaInvokePermission:
    Type: 'AWS::Lambda::Permission'
    Properties:
      FunctionName: !GetAtt MyLambdaFunction.Arn
      Action: 'lambda:InvokeFunction'
      Principal: 'events.amazonaws.com'
      SourceArn: !GetAtt MyCloudWatchEventRule.Arn

AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  Environment:
    Type: String
    Description: The environment to deploy (e.g., local, production)
    AllowedValues:
      - local
      - production
    Default: local

Conditions:
  IsLocal: !Equals [ !Ref Environment, local ]
  IsProduction: !Equals [ !Ref Environment, production ]

Resources:
  MyLambdaFunction:
    Type: 'AWS::Lambda::Function'
    Properties: 
      FunctionName: MyLambdaFunction
      Handler: index.handler
      Role: arn:aws:iam::123456789012:role/service-role/MyLambdaRole
      Runtime: python3.8
      Code:
        S3Bucket: my-bucket
        S3Key: my-lambda-code.zip
      Environment: 
        Variables: 
          CLIENT_ID: 
            Fn::If:
              - IsLocal
              - local-client-id
              - production-client-id
          CLIENT_SECRET: 
            Fn::If:
              - IsLocal
              - local-client-secret
              - production-client-secret
          OTHER_ENV_VAR: 
            Fn::If:
              - IsLocal
              - local-value
              - production-value


Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub my-s3-bucket-${AWS::AccountId}

  MyLambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      Handler: index.handler
      Role: arn:aws:iam::123456789012:role/service-role/MyLambdaRole
      Code:
        S3Bucket: my-lambda-functions
        S3Key: function.zip
      Runtime: nodejs14.x
      FunctionName: MyLambdaFunction

  MyBucketPermission:
    Type: AWS::Lambda::Permission
    Properties: 
      FunctionName: !GetAtt MyLambdaFunction.Arn
      Action: lambda:InvokeFunction
      Principal: s3.amazonaws.com
      SourceAccount: !Sub ${AWS::AccountId}
      SourceArn: !GetAtt MyBucket.Arn

  MyBucketNotification:
    Type: AWS::S3::BucketNotification
    Properties:
      Bucket: !Ref MyBucket
      NotificationConfiguration:
        LambdaConfigurations:
          - Event: s3:ObjectCreated:Put
            Filter:
              S3Key:
                Rules:
                  - Name: prefix
                    Value: my-prefix/
            Function: !GetAtt MyLambdaFunction.Arn





            import json
import requests

def lambda_handler(event, context):
    # Thay thế URL token và API của Volue Insight cùng với client ID và client secret của bạn
    token_url = "https://api.volueinsight.com/token"
    client_id = "YOUR_CLIENT_ID_HERE"
    client_secret = "YOUR_CLIENT_SECRET_HERE"
    
    # Yêu cầu token
    response = requests.post(
        token_url,
        data={
            'grant_type': 'client_credentials',
            'client_id': client_id,
            'client_secret': client_secret
        }
    )

    if response.status_code == 200:
        token = response.json()['access_token']
        
        # Thay thế URL API lấy giá điện
        spot_price_url = "https://api.volueinsight.com/spot-prices"
        headers = {
            "Authorization": f"Bearer {token}"
        }

        # Gọi API lấy giá điện
        spot_price_response = requests.get(spot_price_url, headers=headers)

        if spot_price_response.status_code == 200:
            data = spot_price_response.json()
            return {
                'statusCode': 200,
                'body': json.dumps(data)
            }
        else:
            return {
                'statusCode': spot_price_response.status_code,
                'body': spot_price_response.text
            }
    else:
        return {
            'statusCode': response.status_code,
            'body': response.text
        }

import json
import requests
import pandas as pd
import boto3
from io import StringIO
from datetime import datetime, timedelta

def lambda_handler(event, context):
    # Thay thế URL token và API của Volue Insight cùng với client ID và client secret của bạn
    token_url = "https://api.volueinsight.com/token"
    client_id = "YOUR_CLIENT_ID_HERE"
    client_secret = "YOUR_CLIENT_SECRET_HERE"
    
    # Yêu cầu token
    response = requests.post(
        token_url,
        data={
            'grant_type': 'client_credentials',
            'client_id': client_id,
            'client_secret': client_secret
        }
    )

    if response.status_code == 200:
        token = response.json()['access_token']
        
        # Tính toán ngày mai
        tomorrow = (datetime.utcnow() + timedelta(days=1)).strftime('%Y-%m-%d')
        
        # Thay thế URL API lấy dữ liệu spotex và thêm tham số ngày mai
        spotex_url = f"https://api.volueinsight.com/spotex?date={tomorrow}"
        headers = {
            "Authorization": f"Bearer {token}"
        }

        # Gọi API lấy dữ liệu spotex của ngày mai
        spotex_response = requests.get(spotex_url, headers=headers)

        if spotex_response.status_code == 200:
            data = spotex_response.json()
            
            # Chuyển dữ liệu thành DataFrame
            df = pd.DataFrame(data)

            # Tạo file CSV từ DataFrame
            csv_buffer = StringIO()
            df.to_csv(csv_buffer, index=False)

            # Lưu file CSV vào S3
            s3 = boto3.client('s3')
            bucket_name = 'your-s3-bucket-name'
            file_name = f'spotex_data_{tomorrow}.csv'
            
            s3.put_object(Bucket=bucket_name, Key=file_name, Body=csv_buffer.getvalue())

            return {
                'statusCode': 200,
                'body': json.dumps(f'Successfully created {file_name} in {bucket_name}')
            }
        else:
            return {
                'statusCode': spotex_response.status_code,
                'body': spotex_response.text
            }
    else:
        return {
            'statusCode': response.status_code,
            'body': response.text
        }
