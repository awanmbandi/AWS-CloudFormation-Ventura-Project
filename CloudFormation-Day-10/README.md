
## Steps
- Deploy StackSet Permissions Stack to Admin Account 
- Deploy StackSet Permissions Stack to Target Accounts
- Deploy StackSet Environment
- Deploy StackSet Environment Instances

## Environment Vars
```shell
PROFILE=AdministratorAccount
STACKSET_NAME=dev-env-stackset
REGION_1=us-east-1
REGION_2=us-west-1
ADMIN_ACCOUNT_ID=XXXXXXXXXXXXXX
DEV_JAMES_ACCOUNT_ID=YYYYYYYYYYYYYYY
DEV_SANDRA_ACCOUNT_ID=ZZZZZZZZZZZZZZZ
```

644483461595,213158396549

## Deploy Permission Stack to Admin Account  (SKIP and DEPLOY MANUALLY)
```shell
aws cloudformation create-stack \
  --stack-name dev-stackset-permissions \
  --template-body file://./AWSCloudFormationStackSetAdministrationRole.yml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region $REGION_1 \
  --profile $PROFILE
```

## Deploy Permission Stack to Target Accounts  (SKIP and DEPLOY MANUALLY)
```shell
aws cloudformation create-stack \
  --stack-name dev-stackset-permissions \
  --template-body file://./AWSCloudFormationStackSetExecutionRole.yml \
  --parameters ParameterKey=AdministratorAccountId,ParameterValue=$ADMIN_ACCOUNT_ID,UsePreviousValue=true,ResolvedValue=string \
  --capabilities CAPABILITY_NAMED_IAM \
  --region $REGION_1 \
  --profile $PROFILE_1

aws cloudformation create-stack \
  --stack-name dev-stackset-permissions \
  --template-body file://./AWSCloudFormationStackSetExecutionRole.yml \
  --parameters ParameterKey=AdministratorAccountId,ParameterValue=$ADMIN_ACCOUNT_ID,UsePreviousValue=true,ResolvedValue=string \
  --capabilities CAPABILITY_NAMED_IAM \
  --region $REGION_1 \
  --profile $PROFILE_2
```

## Create StackSet  (CONTINUE FROM HERE WITH THE DEPLOYMENT VIA(CLI))
```shell
aws cloudformation create-stack-set \
  --stack-set-name $STACKSET_NAME \
  --template-body file://./template.yaml \
  --profile $PROFILE
```

## List StackSets
```shell
aws cloudformation list-stack-sets \
  --profile $PROFILE
```

## Add StackSet Instances  (Give It Some Time For The Instances To Come Up)
```shell
aws cloudformation create-stack-instances \
  --stack-set-name $STACKSET_NAME \
  --accounts $DEV_JAMES_ACCOUNT_ID $DEV_SANDRA_ACCOUNT_ID \
  --regions $REGION_1 $REGION_2 \
  --profile $PROFILE \
  --operation-preferences FailureToleranceCount=0,MaxConcurrentCount=1
```

## Update StackSet Changing BucketName
```shell
aws cloudformation update-stack-set \
  --stack-set-name $STACKSET_NAME \
  --template-body file://./template.yaml \
  --region $REGION_1 \
  --profile $PROFILE
```

## Deploy Account Gate Functions for Target Accounts
```shell
aws cloudformation deploy \
  --stack-name dev-account-gate \
  --template-file setup.yaml \
  --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
  --region $REGION_1 \
  --profile $PROFILE_1

aws cloudformation deploy \
  --stack-name dev-account-gate \
  --template-file setup.yaml \
  --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
  --region $REGION_2 \
  --profile $PROFILE_2
```

## Helpful Commands

List Stacks
```shell
aws cloudformation list-stacks \
  --region $REGION_1 \
  --profile $PROFILE
```

Delete Stack
```shell
aws cloudformation delete-stack \
  --stack-name dev-stackset-permissions \
  --region $REGION_1 \
  --profile $PROFILE_1
```

## Helpful Links

- [Custom StackSetsResource](https://github.com/awslabs/aws-cloudformation-templates/blob/master/aws/solutions/StackSetsResource/Templates/stack-set-template.yaml)
