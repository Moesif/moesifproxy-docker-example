Moesif Secure Proxy Example with Docker 

Example `docker-compose.yml` to deploy the Moesif secure proxy and NGINX for SSL termination. 
SSL certificates are automatically created via Let's Encrypt using [https-portal](https://github.com/SteveLTN/https-portal)

[Moesif Secure Proxy](https://www.moesif.com/docs/platform/secure-proxy/) enables you to leverage zero-knowledge security for your data stored in [Moesif API Analytics](https://www.moesif.com/).

## How to run this example

### 1. Create an AWS Key Management Service
Within your AWS Console, create a new [Key Management Service](https://aws.amazon.com/kms/). This handles key generation and storage. 
Because Moesif doesn't have access to your master encryption keys, Moesif and its employees cannot view your event data in plain text.

#### AWS_CUSTOMER_KEY_ID
AWS_CUSTOMER_KEY_ID is the Customer managed key (CMK) in AWS KMS, To find KeyId follow the instructions [here](https://docs.aws.amazon.com/kms/latest/developerguide/find-cmk-id-arn.html)
#### AWS_KMS_REGION
AWS_KMS_REGION is a string representing aws region where AWS KMS is configured. Defaults to 'us-west-2'
#### AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
These are access keys needed to access AWS KMS via api. More information about access keys [here](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys). Make sure that IAM user/role associated with the access keys has permissions to access AWS KMS.
#### AWS_SECURE_PROXY_ROLE_ARN
When Secure proxy is run in AWS,  AWS supports multiple ways to inject the auth credentials directly to a EC2 instance or via Kubernetes service accounts.
In this case pass the IAM role AWS_SECURE_PROXY_ROLE_ARN as environment variable, Secure Proxy will assume this role to gain auth access to AWS KMS. 
Note for Secure Proxy to assume role AWS_SECURE_PROXY_ROLE_ARN, base creds should have been injected already by AWS or K8s service accounts to pods.

Note: (AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY) Or AWS_SECURE_PROXY_ROLE_ARN is needed for Secure Proxy to access AWS KMS

> If you don't have an AWS account, you can create one for free. You can still run the actual docker container in your cloud provider of choice.

### 2. Update DNS provider

Create a domain like `moesif.acmeinc.com` and add a CNAME to your DNS provider that points to the host you're running the example on. 

### 3. Modify docker-compose.yml

You'll need to clone this repo and modify the `docker-compose.yml` file with your correct information.

#### `moesifproxy`

The `moesifproxy` is a image provided by Moesif that encrypts and decrypts your data on the fly using Bring Your Own Key (BYOK). Your master keys are stored in AWS Key Management Service. 

* Set `MOESIF_APPLICATION_ID` to your Application Id which can be found by logging into [Moesif](https://www.moesif.com) and going to API Keys from the top-right menu.
* Set `MOESIF_MANAGEMENT_API_KEY` to your Moesif Management API key which can be found by logging into Moesif and going to API Keys from the top-right menu.
Ensure the key is generated with at least the `create:encrypted_keys`, `read:encrypted_keys` scopes.
* Set `AWS_CUSTOMER_KEY_ID`, `AWS_ACCESS_KEY_ID`, and `AWS_SECRET_ACCESS_KEY` to your credentials from step 1. 

#### `https-portal`

The `https-portal` handles SSL termination so that you can safely expose the secure proxy to the internet. This enables you to leverage features like Moesif's embedded templates 
even with encrypted data. In this case, you should add a record to your DNS provide that points to the secure proxy like `analytics.acmeinc.com`. 
Moesif strongly recommends adding SSL such as via a load balancer in front of the secure proxy like NGINX or HaProxy. 

* Update `DOMAINS` to map your actual domain to the internal service `moesifproxy` like so `moesifproxy.acmeinc.com -> http://moesifproxy:9500`.

### Run the container

Now that your environment variables are set correctly, run the example as follows. 

```bash
docker-compose -f docker-compose.yml up -d
```

## Troubleshooting

### ValueError: Challenge did not pass for

Ensure your domain is reachable by the public internet on both ports 443 and 80. Because a probe to your domain is made quickly after boot, ensure any firewalls or load balancers probes are correctly configured to route traffic to your virtual machine before starting the containers. 
