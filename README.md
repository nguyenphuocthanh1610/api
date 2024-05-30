// Import các module cần thiết từ AWS SDK v3
const { S3Client, GetObjectCommand } = require("@aws-sdk/client-s3");
const { STSClient, AssumeRoleCommand } = require("@aws-sdk/client-sts");
const { fromTemporaryCredentials } = require("@aws-sdk/credential-providers");

// Hàm chính để truy cập S3
async function accessS3WithAssumeRole() {
    // Cấu hình thông tin Assume Role
    const roleToAssume = {
        RoleArn: 'arn:aws:iam::123456789012:role/YourRoleName', // Thay thế bằng ARN của vai trò bạn muốn giả định
        RoleSessionName: 'assume-role-session',
    };

    // Tạo client STS để assume role
    const stsClient = new STSClient({ region: 'us-west-2' });

    // Giả định vai trò và lấy thông tin xác thực tạm thời
    const assumeRoleCommand = new AssumeRoleCommand(roleToAssume);
    const assumedRole = await stsClient.send(assumeRoleCommand);

    // Tạo client S3 với thông tin xác thực tạm thời
    const s3Client = new S3Client({
        region: 'us-west-2',
        credentials: fromTemporaryCredentials({
            params: {
                RoleArn: roleToAssume.RoleArn,
                RoleSessionName: roleToAssume.RoleSessionName,
            },
            clientConfig: { region: 'us-west-2' }, // Cấu hình vùng miền nếu cần
            masterCredentials: stsClient.config.credentials, // Dùng client STS để lấy thông tin xác thực
        }),
    });

    // Thực hiện lệnh truy cập S3 (ví dụ: lấy một đối tượng từ S3)
    try {
        const getObjectParams = {
            Bucket: 'your-bucket-name', // Thay thế bằng tên bucket của bạn
            Key: 'your-object-key', // Thay thế bằng key của đối tượng bạn muốn lấy
        };

        const command = new GetObjectCommand(getObjectParams);
        const data = await s3Client.send(command);

        console.log('Object data:', data);
    } catch (error) {
        console.error('Error accessing S3:', error);
    }
}

// Gọi hàm chính
accessS3WithAssumeRole();
