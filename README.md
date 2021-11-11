A sweet collection of **CloudFormation** templates.

```
aws cloudformation create-stack
    --stack-name <name>
    --template-body <body>
    --parameters ParameterKey=<key>,ParameterValue=<value> ParameterKey=<key>,ParameterValue=<value>
```

# Dependencies


| Stack  |  Dependency |
| ------ | ----------- |
| UserStack | None |
| PolicyStack | UserStack |
| ECRStack | None | 
| VPCStack-$ENV | None | 
| FrontendStack-$ENV | UserStack |
| DNSStack-$ENV | FrontendStack-$ENV |
| RDSStack-$ENV | VPCStack-$ENV | 
| LambdaStack-$ENV | VPCStack-$ENV, ECRStack, UserStack |
| GatewayStack-$ENV | UserStack, LambdaStack-$ENV |

# Steps

A more detailed version of what follows can be found on the [Confluence page](https://makpar.atlassian.net/wiki/spaces/IN/pages/356483073/AWS+Resource+and+CI+CD+Setup+Walk-Thru)

```
cp .sample.env .env
# configure stack names and RDS credentials in .env file 
source .env
./scripts/user-stack
./scripts/policy-stack 
./scripts/frontend-stack --environment <Dev | Prod | Test | Staging> 
./scripts/dns-stack [--dns-exists]
./scripts/vpc-stack --environment <Dev | Prod | Test | Staging>
./scripts/rds-stack --environment <Dev | Prod | Test | Staging>
# Pass RDS Host Url to SecretManager
./scrips/rds-host-secret --environment <Dev | Prod | Test | Staging>
./scripts/ecr-stack --components <one | two | three | four | five>
# Build images and push to ECR; use ./scripts/docker/build-images from lambda-pipeline repo
./scripts/lambda-stack --components <one | two | three | four | five> --environment <Dev | Prod | Test | Staging>
./scripts/gateway-stack --environment <Dev | Prod | Test | Staging>
```

NOTE: all scripts have an optional argument ``--action`` with allowable values of `create` or `update`. If `update` is passed through the ``--action`` flag, the script will update the current stack instead of creating a new one.

# Notes

1. When creating users through a **CloudFormation** template, you must explicitly tell **CloudFormation** that it's okay to create new users with new permissions. See [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_CreateStack.html). Essentially, when you are creating a stack that involves creating new users, you have to pass in the following flag,

```
aws cloudformation create-stack
    --stack-name UserStack
    --template-body file://path/to/template.yml
    --capabilities CAPABILITY_NAMED_IAM
```

2. In betweens standing up the **ECR** stack and the **Lambda** stack, the images for the **lambdas** will need initialized and pushed to the repo. **lambda** needs the image to exist before it can successfull deploy.

3. After the **RDS** stack goes up, the host URL for the RDS instance will need inserted into the AWS **SecretsManager**. Use the script,

```
./scripts/secret-host-rds --environment < Dev | Staging | Test | Prod >
```

Note: The username and password secrets for the RDS are created during the `rds-stack` script, but the host URL secret creation cannot be automated since it doesn't exist until the RDS stack is provisioned.

4. If an API key needs delivered to the **Lambda** function environment, before the **Lambda** stack goes up, update the **API_KEY** environment variable in *.env* environment file and use the script,

```
./scripts/secret-api-key --environemtn
```

# Documentation
## CloudFormation
### CLI
- [create-stack](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/create-stack.html)
- [delete-stack](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/delete-stack.html)
- [describe-stacks](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/describe-stacks.html)
- [list-stacks](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/list-stacks.html)

### Template References
- [API Gateway](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_ApiGateway.html)
- [ECR](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_ECR.html)
- [IAM](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_IAM.html)
- [Lambda](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_Lambda.html)
