Bước 1: Tạo cấu trúc thư mục dự án
Giả sử bạn có dự án Python với cấu trúc thư mục sau:
my-python-app/
├── app/
│   └── main.py
├── Dockerfile
├── docker-compose.yml
└── requirements.txt
Bước 2: Tạo Dockerfile
Tạo file Dockerfile để định nghĩa môi trường chạy ứng dụng Python:
# Use the official Python image from the Docker Hub
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file into the container
COPY requirements.txt .

# Install any dependencies
RUN pip install -r requirements.txt

# Copy the rest of the application code
COPY . .

# Command to run the application
CMD ["python", "main.py"]
Bước 3: Tạo docker-compose.yml
Tạo file docker-compose.yml để định nghĩa dịch vụ và cài đặt đồng bộ hóa mã nguồn:
version: '3'
services:
  app:
    build: .
    volumes:
      - ./app:/app
    ports:
      - "5000:5000"
    command: python /app/main.py
Bước 4: Tạo requirements.txt
Tạo file requirements.txt để liệt kê các thư viện cần thiết cho ứng dụng của bạn:
watchdog
Bước 5: Tạo script theo dõi thay đổi
Tạo script Python sử dụng watchdog để theo dõi các thay đổi trong mã nguồn và tự động rebuild container khi cần thiết.

Tạo file watcher.py trong thư mục dự án:
import os
import time
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class ChangeHandler(FileSystemEventHandler):
    def on_any_event(self, event):
        if event.event_type in ('created', 'modified', 'deleted'):
            os.system('docker-compose up --build -d')

if __name__ == "__main__":
    path = "./app"
    event_handler = ChangeHandler()
    observer = Observer()
    observer.schedule(event_handler, path, recursive=True)
    observer.start()
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
    observer.join()
Bước 6: Chạy Docker Compose
Mở terminal và điều hướng đến thư mục dự án của bạn, sau đó chạy lệnh sau để khởi động dịch vụ Docker:

docker-compose up --build -d
Bước 7: Chạy script theo dõi thay đổi
Trong một terminal khác, điều hướng đến thư mục dự án và chạy script watcher.py:
python watcher.py
