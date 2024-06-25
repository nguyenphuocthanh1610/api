zip lambda_function.zip lambda_function.py
aws s3 cp lambda_function.zip s3://your-bucket-name/lambda_function.zip

#lambda.yml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyLambdaFunction:
    Type: 'AWS::Lambda::Function'
    Properties:
      Handler: lambda_function.lambda_handler
      Role: arn:aws:iam::your-account-id:role/your-lambda-execution-role
      Code:
        S3Bucket: your-bucket-name
        S3Key: lambda_function.zip
      Runtime: python3.8
      Timeout: 15
      MemorySize: 128


#aws cloudformation create-stack --stack-name my-lambda-stack --template-body file://lambda.yml --capabilities CAPABILITY_IAM
#aws cloudformation describe-stacks --stack-name my-lambda-stack
------------------
Tạo Thư mục cho Layer
mkdir python
Cài Đặt Thư Viện
Sử dụng pip để cài đặt các thư viện mà bạn muốn bao gồm trong layer. Chạy lệnh sau trong thư mục python:
pip install requests -t python
Đóng Gói Thư Mục Thành Tệp ZIP
zip -r python_layer.zip python
Tải Tệp ZIP Lên S3
aws s3 cp python_layer.zip s3://your-bucket-name/python_layer.zip
ạo Tệp CloudFormation
Tạo một tệp CloudFormation, ví dụ layer.yml, với nội dung sau:
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyLambdaLayer:
    Type: 'AWS::Lambda::LayerVersion'
    Properties:
      LayerName: 'MyPythonLayer'
      Description: 'A Lambda layer with Python libraries'
      Content:
        S3Bucket: your-bucket-name
        S3Key: python_layer.zip
      CompatibleRuntimes:
        - python3.8
        - python3.9
        - python3.10
      LicenseInfo: 'MIT'
Triển Khai CloudFormation Stack
aws cloudformation create-stack --stack-name my-layer-stack --template-body file://layer.yml --capabilities CAPABILITY_NAMED_IAM
Trong tệp CloudFormation của Lambda function (lambda.yml), thêm thuộc tính Layers vào phần Properties của Lambda function:
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyLambdaFunction:
    Type: 'AWS::Lambda::Function'
    Properties:
      Handler: lambda_function.lambda_handler
      Role: arn:aws:iam::your-account-id:role/your-lambda-execution-role
      Code:
        S3Bucket: your-bucket-name
        S3Key: lambda_function.zip
      Runtime: python3.8
      Timeout: 15
      MemorySize: 128
      Layers:
        - !Ref MyLambdaLayer

aws cloudformation update-stack --stack-name my-lambda-stack --template-body file://lambda.yml --capabilities CAPABILITY_IAM
