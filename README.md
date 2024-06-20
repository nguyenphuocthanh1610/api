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
