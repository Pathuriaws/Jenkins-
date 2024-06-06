To implement the AWS STS (Security Token Service) for assuming an IAM role without using long-term AWS access keys, you can use the aws-sdk library to assume a role and retrieve temporary credentials. Here’s how you can modify your code to use STS for assuming a role:

	1.	Install the AWS SDK:
Make sure you have the AWS SDK installed in your project. You can install it using npm if you haven’t already:

npm install aws-sdk


	2.	Update your code to use STS:
Here’s an example of how you can update your existing code to use AWS STS to assume a role and then access the secrets manager:

import { STS, SecretsManager } from 'aws-sdk';

async function getTemporaryCredentials() {
  const sts = new STS({
    region: process.env.AWS_SECRET_MANAGER_REGION,
  });

  const assumedRole = await sts.assumeRole({
    RoleArn: process.env.AWS_ASSUME_ROLE_ARN!,
    RoleSessionName: 'session1'
  }).promise();

  return {
    accessKeyId: assumedRole.Credentials?.AccessKeyId!,
    secretAccessKey: assumedRole.Credentials?.SecretAccessKey!,
    sessionToken: assumedRole.Credentials?.SessionToken!,
  };
}

async function getPrivateKey() {
  const temporaryCredentials = await getTemporaryCredentials();

  const secretsManagerClient = new SecretsManager({
    region: process.env.AWS_SECRET_MANAGER_REGION,
    accessKeyId: temporaryCredentials.accessKeyId,
    secretAccessKey: temporaryCredentials.secretAccessKey,
    sessionToken: temporaryCredentials.sessionToken,
  });

  const secretsManagerResult: any = await secretsManagerClient.getSecretValue({
    SecretId: process.env.AWS_SECRET_MANAGER_KEY_NAME || '',
  }).promise();

  if (secretsManagerResult && secretsManagerResult.SecretString) {
    return JSON.parse(secretsManagerResult.SecretString)['SENDER_PK'];
  } else {
    return false;
  }
}

// Example usage
getPrivateKey().then(privateKey => {
  console.log('Private Key:', privateKey);
}).catch(error => {
  console.error('Error:', error);
});



In this code:

	•	getTemporaryCredentials: This function uses the STS service to assume a role specified by AWS_ASSUME_ROLE_ARN. It returns the temporary credentials (access key, secret access key, and session token) needed to authenticate subsequent AWS SDK calls.
	•	getPrivateKey: This function uses the temporary credentials obtained from getTemporaryCredentials to create a new SecretsManager client. It then retrieves the secret value and returns the private key.

Environment Variables

Ensure you have the following environment variables set in your environment or a .env file:

	•	AWS_SECRET_MANAGER_REGION: The region of the Secrets Manager.
	•	AWS_ASSUME_ROLE_ARN: The ARN of the role to assume.
	•	AWS_SECRET_MANAGER_KEY_NAME: The name of the secret in Secrets Manager.

By using this method, you avoid using long-term access keys and rely on temporary credentials obtained through STS, which is a more secure and flexible approach.
