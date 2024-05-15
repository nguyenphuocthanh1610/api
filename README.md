
Sao chép mã
const { S3Client, CopyObjectCommand } = require("@aws-sdk/client-s3");
const { NodeHttpHandler } = require("@aws-sdk/node-http-handler");

// Khởi tạo S3 client với các tùy chọn HTTP
const s3Client = new S3Client({
  region: "your-region",
  credentials: {
    accessKeyId: "your-access-key-id",
    secretAccessKey: "your-secret-access-key",
  },
  requestHandler: new NodeHttpHandler({
    connectionTimeout: 5000,  // Thời gian chờ kết nối (ms)
    socketTimeout: 10000      // Thời gian chờ cho toàn bộ request (ms)
  })
});
