const { STSClient, AssumeRoleCommand } = require("@aws-sdk/client-sts");

const stsClient = new STSClient();

function assumeRole() {
    const command = new AssumeRoleCommand({
        RoleArn: "arn:aws:iam::123456789012:role/ExampleRole",
        RoleSessionName: "example-session"
    });

    stsClient.send(command)
        .then((response) => {
            console.log("Assume Role Successful:", response);
        })
        .catch((error) => {
            console.error("Error assuming role:", error);
        });
}

assumeRole();
