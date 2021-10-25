A sweet collection of **CloudFormation** templates.

```
aws cloudformation create-stack
    --stack-name <name>
    --template-body <body>
    --parameters ParameterKey=<key>,ParameterValue=<value> ParameterKey=<key>,ParameterValue=<value>
```

# Dependencies

#

| Stack  |  Dependency |
| ------ | ----------- |
| UsersStack | None |
| VPCStack-$ENV | None | 
| ECRStack | UsersStack | 
| RDSStack-$ENV | VPCStack-$ENV | 
| LambdaStack-$ENV | VPCStack-$ENV, ECRStack, UsersStack |
| GatewayStack-$ENV | UsersStack, LambdaStack-$ENV |

# Steps


```
cp .sample.env .env
source .env
./scripts/users-stack
./scripts/vpc-stack --environment <Dev | Prod | Test>
./scripts/rds-stack --environment <Dev | Prod | Test>
./scrips/rds-host-secret --environment <Dev | Prod | Test>
./scripts/ecr-stack --components <one | two | three | four >
# Build images and push to ECR; use ./scripts/docker/build-images from lambda-pipeline repo
./scripts/lambda-stack --components <one | two | three | four> --environment <Dev | Prod | Test>
./scripts/gateway-stack --environment <Dev | Prod | Test>
```

# Notes

1. When creating users through a **CloudFormation** template, you must explicitly tell **CloudFormation** that it's okay to create new users with new permissions. See [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_CreateStack.html). Essentially, when you are creating a stack that involves creating new users, you have to pass in the following flag,

```
aws cloudformation create-stack
    --stack-name UserStack
    --template-body file://path/to/template.yml
    --capabilities CAPABILITY_NAMED_IAM
```

2. In betweens standing up the **ECR** stack and the **Lambda** stack, the images for the **lambdas** will need initialized and pushed to the repo. **lambda** needs the image to exist before it can successfull deploy.

3. the **ECR** stack and the **User** stack are independent of the environment being provisioned. The differences in the environment images is managed through tags in the **ECR** repo, while users and roles are global resources available in all environments.

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
