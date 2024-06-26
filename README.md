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
