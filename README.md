AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  Environment:
    Description: "Deployment environment"
    Type: String
    Default: "local"
    AllowedValues:
      - local
      - production
      - staging
    ConstraintDescription: "must specify local, production, or staging."

Conditions:
  IsLocal: !Equals [!Ref Environment, "local"]
  IsProduction: !Equals [!Ref Environment, "production"]
  IsStaging: !Equals [!Ref Environment, "staging"]

Resources:
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

  MyLambdaFunction:
    Type: 'AWS::Lambda::Function'
    Properties:
      FunctionName: !If 
        - IsLocal
        - "abc"
        - !If 
          - IsStaging
          - "xyz"
          - "def"
      Handler: 'lambda_function.lambda_handler'
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        S3Bucket: your-bucket-name
        S3Key: lambda_function.zip
      Runtime: 'python3.8'
      Timeout: 15
      MemorySize: 128
      Environment:
        Variables:
          DATABASE_URL: !If 
            - IsLocal
            - "http://localhost:5432/mydb"
            - "https://prod-db.amazonaws.com/mydb"
          LOG_LEVEL: !If
            - IsLocal
            - "DEBUG"
            - "INFO"

Outputs:
  LambdaFunctionName:
    Description: 'Name of the created Lambda function'
    Value: !Ref MyLambdaFunction
aws cloudformation create-stack --stack-name my-lambda-stack-local --template-body file://template.yml --parameters ParameterKey=Environment,ParameterValue=local --capabilities CAPABILITY_IAM
