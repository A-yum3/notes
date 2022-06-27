## 実現したいこと

AWS SAMでLambdaに勝手にデプロイしてくれたら嬉しいので、実現を目指す。

## 前提条件

- `samconfig.toml`がある
- `template.yml`がある

## 出来上がったyml

```config.yaml
version: 2.1

orbs:
  aws-cli: circleci/aws-cli@3.1.1
  sam: circleci/aws-sam-serverless@3.2.0

executors:
  node:
    docker:
      - image: cimg/node:18.2.0
#    resource_class: small
    resource_class: large

commands:
  aws-auth:
    description: AWSとOIDCで認証を行い、認証情報を環境変数にexportする
    steps:
      - aws-cli/install
      - run: |
          aws_sts_credentials=$(aws sts assume-role-with-web-identity \
          --role-arn ${AWS_ROLE_ARN} \
          --web-identity-token ${CIRCLE_OIDC_TOKEN} \
          --role-session-name "circleci-oidc" \
          --duration-seconds 900 \
          --query "Credentials" \
          --output "json")
          echo export AWS_ACCESS_KEY_ID="$(echo $aws_sts_credentials | jq -r '.AccessKeyId')" >> $BASH_ENV
          echo export AWS_SECRET_ACCESS_KEY="$(echo $aws_sts_credentials | jq -r '.SecretAccessKey')" >> $BASH_ENV
          echo export AWS_SESSION_TOKEN="$(echo $aws_sts_credentials | jq -r '.SessionToken')" >> $BASH_ENV
          source $BASH_ENV

  deploy-dev:
    description: SAMを利用してdev環境のLambdaにデプロイを行なう
    steps:
      - sam/install
      - run:
          command: |
            sam deploy \
            --config-file samconfig.dev.toml \
            --no-confirm-changeset

jobs:
  deploy-dev:
    executor: node
    steps:
      - checkout
      - aws-auth
      - deploy-dev

workflows:
  version: 2
  deploy:
    jobs:
      - deploy-dev:
          context:
            - comsbi_lambda_deploy


```

## はまりどころ

### OpenID Connectを使ってCircleCIとAWSを連携させる

---

参考資料
[CircleCIがOpenID ConnectをサポートしたのでAWSと連携させてJWTを使用したAssumeRoleを試してみた](https://dev.classmethod.jp/articles/circleci-supported-oidc-so-i-tried-linking-it-with-aws/)
[CircleCIとAWSをOpenID Connect 認証で連携する](https://qiita.com/suzucir/items/5b76ae7d80944d5d3c85)

---

参考資料をもとに、Contextを作成し、IDプロバイダIAMを作成する。

IAMを作成したらRoleを割り当て、ここで作成したContextに`AWS_DEFAULT_REGION`と`AWS_ROLE_ARN`を設定する。

### SAMからLambdaにDeployする最小ポリシーが難しい

AWS初心者に優しくないポリシーしてました。

---
参考資料
[AWS Lambdaのデプロイに必要なIAMポリシーについて](https://qiita.com/ho-rai/items/1a3a872cd85c414d9c24)

---

上記の参考資料から

#### CloudFormation

DescribeStackEvents
DescribeStackSetOperation
CreateChangeSet
DescribeChangeSet
ExecuteChangeSet
GetTemplateSummary
DescribeStacks

-> すべてのリソース（stackのregion指定するだけでもダメでした）

#### IAM

GetRole
PassRole
DetachRolePolicy
CreateRole
DeleteRole
AttachRolePolicy
UpdateRole
PutRolePolicy
CreateUser

-> すべてのリソース

#### S3

GetBucketTagging
GetBucketWebsite
GetJobTagging
GetObjectVersionTagging
RestoreObject
GetBucketAcl
GetBucketNotification
GetBucketPolicy
GetObjectVersionTorrent
PutObject
GetObject
GetIntelligentTieringConfiguration
GetObjectTagging
GetMetricsConfiguration
GetBucketLocation
GetObjectVersion

-> Bucket name指定くらいまで

#### Lambda

GetFunction
CreateFunction
UpdateFunctionCode
DeleteFunction
AddPermission
ListTags （参考資料にないけど）

-> Function指定くらいまで

ここがとてもつらかった。

## まとめ

CircleCIよりもAWSのポリシーのがとっても苦労しました。