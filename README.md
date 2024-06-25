AWSTemplateFormatVersion: '2010-09-09'
Resources:
  # Định nghĩa IAM Role cho Lambda
  LambdaExecutionRole:
    Type: 'AWS::IAM::Role'
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: 'Allow'
            Principal:
              Service: 'lambda.amazonaws.com'
            Action: 'sts:AssumeRole'
      Policies:
        - PolicyName: 'LambdaS3FullAccessPolicy'
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: 'Allow'
                Action:
                  - 's3:*'
                Resource: '*'
        - PolicyName: 'LambdaBasicExecutionPolicy'
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: 'Allow'
                Action:
                  - 'logs:CreateLogGroup'
                  - 'logs:CreateLogStream'
                  - 'logs:PutLogEvents'
                Resource: 'arn:aws:logs:*:*:*'

  # Định nghĩa Lambda function
  MyLambdaFunction:
    Type: 'AWS::Lambda::Function'
    Properties:
      FunctionName: 'MyCustomLambdaFunctionName'
      Handler: 'lambda_function.lambda_handler'
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        S3Bucket: your-bucket-name
        S3Key: lambda_function.zip
      Runtime: 'python3.8'
      Timeout: 15
      MemorySize: 128

Outputs:
  LambdaFunctionName:
    Description: 'Name of the created Lambda function'
    Value: !Ref MyLambdaFunction
  LambdaRoleArn:
    Description: 'ARN of the Lambda execution role'
    Value: !GetAtt LambdaExecutionRole.Arn
