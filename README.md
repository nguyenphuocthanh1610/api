const { S3Client, CopyObjectCommand } = require("@aws-sdk/client-s3");

// Cấu hình thông tin xác thực AWS
const s3Client = new S3Client({
  region: "your-region",
  credentials: {
    accessKeyId: "your-access-key-id",
    secretAccessKey: "your-secret-access-key",
  },
});

// Thông tin nguồn và đích
const sourceBucket = "abc";
const sourceKey = "1/your-filename.ext"; // Đường dẫn file nguồn
const destinationKey = "2/your-filename.ext"; // Đường dẫn file đích

const copyObject = async () => {
  try {
    // Lệnh copy object
    const copyParams = {
      Bucket: sourceBucket,
      CopySource: `${sourceBucket}/${sourceKey}`, // Đường dẫn nguồn: bucket/sourceKey
      Key: destinationKey, // Đường dẫn đích
    };

    // Thực thi lệnh copy
    const command = new CopyObjectCommand(copyParams);
    const response = await s3Client.send(command);
    console.log("File copied successfully:", response);
  } catch (err) {
    console.error("Error copying file:", err);
  }
};

// Gọi hàm copyObject
copyObject();
