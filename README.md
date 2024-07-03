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
