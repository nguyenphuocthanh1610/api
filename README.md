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

