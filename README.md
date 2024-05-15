// Import AWS SDK và cấu hình
const { S3Client, ListObjectsCommand } = require("@aws-sdk/client-s3");
const { fromIni } = require("@aws-sdk/credential-provider-ini");

// Tạo một phiên S3Client
const s3Client = new S3Client({
    credentials: fromIni({ profile: "your_profile_name" }), // Thay thế bằng tên profile của bạn trong file ~/.aws/credentials
    region: "your_region" // Thay thế bằng khu vực S3 của bạn
});

// Hàm để liệt kê các đối tượng trong một bucket
async function listObjects(bucketName) {
    try {
        // Tạo lệnh để liệt kê đối tượng
        const command = new ListObjectsCommand({ Bucket: bucketName });

        // Thực thi lệnh và chờ kết quả
        const response = await s3Client.send(command);

        // Lặp qua danh sách các đối tượng và in ra tên của chúng
        response.Contents.forEach(object => {
            console.log(object.Key);
        });
    } catch (error) {
        console.error("Error listing objects: ", error);
    }
}

// Gọi hàm để liệt kê các đối tượng trong bucket cụ thể
listObjects("your_bucket_name"); // Thay thế bằng tên bucket của bạn
----------------
// Import AWS SDK và cấu hình
const { S3Client, GetObjectCommand } = require("@aws-sdk/client-s3");
const { fromIni } = require("@aws-sdk/credential-provider-ini");
const { Readable } = require("stream");
const fs = require("fs");

// Tạo một phiên S3Client
const s3Client = new S3Client({
    credentials: fromIni({ profile: "your_profile_name" }), // Thay thế bằng tên profile của bạn trong file ~/.aws/credentials
    region: "your_region" // Thay thế bằng khu vực S3 của bạn
});

// Hàm để lấy một đối tượng từ S3 và lưu vào một tệp cục bộ
async function getObject(bucketName, objectKey, filePath) {
    try {
        // Tạo lệnh để lấy đối tượng từ bucket và key cụ thể
        const command = new GetObjectCommand({ Bucket: bucketName, Key: objectKey });

        // Thực thi lệnh và chờ kết quả
        const response = await s3Client.send(command);

        // Tạo một stream để ghi dữ liệu vào tệp cục bộ
        const fileStream = fs.createWriteStream(filePath);

        // Tạo một stream đọc từ phản hồi S3
        const bodyStream = response.Body;

        // Pipe dữ liệu từ stream đến tệp cục bộ
        bodyStream.pipe(fileStream);

        // Đợi cho quá trình ghi hoàn thành
        await new Promise((resolve, reject) => {
            fileStream.on("finish", resolve);
            fileStream.on("error", reject);
        });

        console.log(`Object downloaded successfully to ${filePath}`);
    } catch (error) {
        console.error("Error getting object: ", error);
    }
}

// Gọi hàm để lấy một đối tượng từ S3 và lưu vào tệp cục bộ
getObject("your_bucket_name", "your_object_key", "local_file_path"); // Thay thế bằng thông tin của bạn
