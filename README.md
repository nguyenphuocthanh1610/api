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
