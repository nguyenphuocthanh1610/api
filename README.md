const AWS = require('aws-sdk');

exports.handler = async (event) => {
    const sts = new AWS.STS();
    
    try {
        const data = await sts.assumeRole({
            RoleArn: 'arn:aws:iam::123456789012:role/S3AccessRole',
            RoleSessionName: 'session-name'
        }).promise();
        
        const credentials = {
            accessKeyId: data.Credentials.AccessKeyId,
            secretAccessKey: data.Credentials.SecretAccessKey,
            sessionToken: data.Credentials.SessionToken
        };
        
        const s3 = new AWS.S3({ credentials });

        const params = {
            Bucket: 'your-bucket-name',
            Key: 'your-object-key'
        };

        const s3Data = await s3.getObject(params).promise();
        console.log(s3Data.Body.toString('utf-8'));

        return {
            statusCode: 200,
            body: s3Data.Body.toString('utf-8')
        };
    } catch (err) {
        console.error('Access denied', err);
        return {
            statusCode: 500,
            body: JSON.stringify('Access denied')
        };
    }
};
