# cloudformation-iam

- IAM User や IAM Policy を検証するための CloudFormation テンプレート
- IAM User のログインパスワードや、アクセスキー・シークレットキーをサクッと取得できる
- template.yaml で IAM Policy を書き換えて適宜動作を検証すれば良い


# useful commands

- Mac での実行を想定
- AWS CLI, jq をセットアップしておく


```bash
# シェル変数セットアップ
TEMPLATE=template.yaml
STACK_NAME=my-cloudformation-iam

# テンプレートの検証とデプロイ
aws cloudformation validate-template --template-body file://./$TEMPLATE
aws cloudformation deploy --template-file $TEMPLATE --stack-name $STACK_NAME --capabilities CAPABILITY_IAM

# CloudFormation Stack の出力を取得
STACK_OUTPUT=stack-output.json
aws cloudformation describe-stacks --stack-name $STACK_NAME | jq ".Stacks[0].Outputs[]" > $STACK_OUTPUT

# 出力からアクセスキー取得
cat $STACK_OUTPUT | jq --raw-output ". | select(.OutputKey == \"IAMUserAccessKey\") | .OutputValue"

# 出力からシークレットキー取得
cat $STACK_OUTPUT | jq --raw-output ". | select(.OutputKey == \"IAMUserSecretKey\") | .OutputValue"

# AWS CLI の profile をセットアップして動作確認
PROFILE_NAME=my-prof-$(date +%s)
aws configure --profile $PROFILE_NAME
aws configure list --profile $PROFILE_NAME
aws --profile $PROFILE_NAME sts get-caller-identity
aws --profile $PROFILE_NAME ec2 describe-regions
aws --profile $PROFILE_NAME ec2 describe-availability-zones

# 出力からログインパスワード取得
$(cat $STACK_OUTPUT | jq --raw-output ". | select(.OutputKey == \"GetSecretValueByCLI\") | .OutputValue") | jq --raw-output "fromjson | .password"

# IAM User 名取得
aws cloudformation describe-stack-resources --stack-name $STACK_NAME | jq --raw-output ".StackResources[] | select(.ResourceType == \"AWS::IAM::User\") | .PhysicalResourceId"

# マネジメントコンソールのログイン URL 生成
echo https://$(aws --profile $PROFILE_NAME sts get-caller-identity --query "Account" --output text).signin.aws.amazon.com/console

# クリーンアップ
aws configure --profile $PROFILE_NAME set aws_access_key_id ""
aws configure --profile $PROFILE_NAME set aws_secret_access_key ""
aws cloudformation delete-stack --stack-name $STACK_NAME
```


# useful links

Actions, resources, and condition keys for AWS services - Service Authorization Reference  
https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html