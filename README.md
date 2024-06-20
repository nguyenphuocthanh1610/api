pip install boto3
import json
import csv
import boto3
from datetime import datetime
from io import StringIO

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Giả định rằng dữ liệu JSON được truyền trong 'body' của sự kiện
    data = json.loads(event['body'])
    
    # Chuẩn bị dữ liệu CSV
    csv_buffer = StringIO()
    csv_writer = csv.writer(csv_buffer)

    # Viết tiêu đề CSV
    csv_writer.writerow(['first name', 'last name', 'DOB'])
    
    # Viết các dòng dữ liệu CSV
    for item in data:
        csv_writer.writerow([item.get('first name'), item.get('last name'), item.get('DOB')])

    # Tạo tên file với định dạng ngày hiện tại yyyymmdd.csv
    filename = datetime.now().strftime("%Y%m%d") + ".csv"

    # Upload file CSV lên S3
    bucket_name = 'your-s3-bucket-name'
    s3.put_object(Bucket=bucket_name, Key=filename, Body=csv_buffer.getvalue())

    return {
        'statusCode': 200,
        'body': json.dumps('File uploaded successfully')
    }













    import json
import csv
import boto3
from datetime import datetime
import os

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Giả định rằng dữ liệu JSON được truyền trong 'body' của sự kiện
    data = json.loads(event['body'])
    
    # Tạo tên file với định dạng ngày hiện tại yyyymmdd.csv
    filename = datetime.now().strftime("%Y%m%d") + ".csv"
    file_path = f"/tmp/{filename}"

    # Chuẩn bị dữ liệu CSV và lưu vào file tạm thời
    with open(file_path, mode='w', newline='') as file:
        csv_writer = csv.writer(file)
        
        # Viết tiêu đề CSV
        csv_writer.writerow(['first name', 'last name', 'DOB'])
        
        # Viết các dòng dữ liệu CSV
        for item in data:
            csv_writer.writerow([item.get('first name'), item.get('last name'), item.get('DOB')])

    # Upload file CSV lên S3
    bucket_name = 'your-s3-bucket-name'
    s3.upload_file(file_path, bucket_name, filename)

    return {
        'statusCode': 200,
        'body': json.dumps('File uploaded successfully')
    }






import json
import csv
import boto3
from datetime import datetime
from io import StringIO

# Cấu hình thông tin xác thực
access_key = 'your-access-key'
secret_key = 'your-secret-key'
region_name = 'your-region'  # Ví dụ: 'us-west-2'
bucket_name = 'your-s3-bucket-name'

# Tạo client S3 với thông tin xác thực
s3 = boto3.client(
    's3',
    aws_access_key_id=access_key,
    aws_secret_access_key=secret_key,
    region_name=region_name
)

def create_and_upload_csv(data):
    # Tạo tên file với định dạng ngày hiện tại yyyymmdd.csv
    filename = datetime.now().strftime("%Y%m%d") + ".csv"

    # Chuẩn bị dữ liệu CSV
    csv_buffer = StringIO()
    csv_writer = csv.writer(csv_buffer)

    # Viết tiêu đề CSV
    csv_writer.writerow(['first name', 'last name', 'DOB'])
    
    # Viết các dòng dữ liệu CSV
    for item in data:
        csv_writer.writerow([item.get('first name'), item.get('last name'), item.get('DOB')])

    # Đưa con trỏ của csv_buffer về đầu
    csv_buffer.seek(0)

    # Upload file CSV lên S3
    s3.put_object(Bucket=bucket_name, Key=filename, Body=csv_buffer.getvalue())

    print(f"File {filename} uploaded successfully to {bucket_name}.")

def lambda_handler(event, context):
    # Giả định rằng dữ liệu JSON được truyền trong 'body' của sự kiện
    data = json.loads(event['body'])
    
    # Gọi hàm tạo và upload CSV
    create_and_upload_csv(data)

    return {
        'statusCode': 200,
        'body': json.dumps('File uploaded successfully')
    }

# Để chạy hàm này cục bộ, bạn có thể thử nghiệm với dữ liệu giả định
if __name__ == "__main__":
    test_event = {
        'body': json.dumps([
            {'first name': 'John', 'last name': 'Doe', 'DOB': '1990-01-01'},
            {'first name': 'Jane', 'last name': 'Doe', 'DOB': '1992-02-02'}
        ])
    }
    lambda_handler(test_event, None)

