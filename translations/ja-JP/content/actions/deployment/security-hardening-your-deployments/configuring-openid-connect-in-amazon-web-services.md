---
title: アマゾン ウェブ サービスでの OpenID Connect の構成
shortTitle: Configuring OpenID Connect in Amazon Web Services
intro: ワークフロー内で OpenID Connect を使用して、アマゾン ウェブ サービスで認証を行います。
miniTocMaxHeadingLevel: 3
versions:
  fpt: '*'
  ghae: issue-4856
  ghec: '*'
  ghes: '>=3.5'
type: tutorial
topics:
  - Security
ms.openlocfilehash: 5ac1a902bb9ef397fa6fa157ea58496d57ffd231
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2022
ms.locfileid: '146171854'
---
{% data reusables.actions.enterprise-beta %} {% data reusables.actions.enterprise-github-hosted-runners %}

## 概要

OpenID Connect (OIDC) を使うと、{% data variables.product.prodname_actions %} ワークフローでは、有効期間の長い {% data variables.product.prodname_dotcom %} シークレットとしてアマゾン ウェブ サービス (AWS) 資格情報を格納しなくても、AWS 内のリソースにアクセスできます。 

このガイドでは、{% data variables.product.prodname_dotcom %} の OIDC をフェデレーション ID として信頼するように AWS を構成する方法と、トークンを使って AWS に対する認証とリソースへのアクセスを行う [`aws-actions/configure-aws-credentials`](https://github.com/aws-actions/configure-aws-credentials) のワークフロー例を示します。

## 前提条件

{% data reusables.actions.oidc-link-to-intro %}

{% data reusables.actions.oidc-security-notice %}

## AWS への ID プロバイダーの追加

{% data variables.product.prodname_dotcom %} OIDC プロバイダーを IAM に追加するには、[AWS のドキュメント](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html)を参照してください。

- プロバイダー URL の場合: {% ifversion ghes %}`https://HOSTNAME/_services/token`{% else %}`https://token.actions.githubusercontent.com`{% endif %} を使います
- "Audience" の場合: [公式のアクション](https://github.com/aws-actions/configure-aws-credentials)を使っている場合は、`sts.amazonaws.com` を使います。

### ロールと信頼ポリシーの構成

IAM でロールと信頼を構成するには、AWS のドキュメントの「[ロールの想定](https://github.com/aws-actions/configure-aws-credentials#assuming-a-role)」と「[Web ID または OpenID 接続フェデレーションのためのロールの作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html)」を参照してください。

信頼関係を編集して、`sub` フィールドを検証条件に追加します。 次に例を示します。

```json{:copy}
"Condition": {
  "StringEquals": {
    "{% ifversion ghes %}HOSTNAME/_services/token{% else %}token.actions.githubusercontent.com{% endif %}:aud": "sts.amazonaws.com",
    "{% ifversion ghes %}HOSTNAME/_services/token{% else %}token.actions.githubusercontent.com{% endif %}:sub": "repo:octo-org/octo-repo:ref:refs/heads/octo-branch"
  }
}
```

## {% data variables.product.prodname_actions %} ワークフローを更新する

OIDC のワークフローを更新するには、YAML に 2 つの変更を行う必要があります。
1. トークンのアクセス許可設定を追加します。
2. この [`aws-actions/configure-aws-credentials`](https://github.com/aws-actions/configure-aws-credentials) アクションを使用して、OIDC トークン (JWT) をクラウド アクセス トークンと交換します。

### アクセス許可設定の追加

 {% data reusables.actions.oidc-permissions-token %}

### アクセス トークンの要求

この `aws-actions/configure-aws-credentials` アクションを使うと、{% data variables.product.prodname_dotcom %} OIDC プロバイダーから JWT を受け取り、アクセス トークンを AWS に要求します。 詳細については、AWS の[ドキュメント](https://github.com/aws-actions/configure-aws-credentials)を参照してください。

- `<example-bucket-name>`: ここに S3 バケットの名前を追加します。
- `<role-to-assume>`: 例を AWS ロールに置き換えます。
- `<example-aws-region>`: ここに AWS リージョンの名前を追加します。

```yaml{:copy}
# Sample workflow to access AWS resources when workflow is tied to branch
# The workflow Creates static website using aws s3
name: AWS example workflow
on:
  push
env:
  BUCKET_NAME : "<example-bucket-name>"
  AWS_REGION : "<example-aws-region>"
# permission can be added at job level or workflow level    
permissions:
      id-token: write
      contents: read    # This is required for actions/checkout
jobs:
  S3PackageUpload:
    runs-on: ubuntu-latest
    steps:
      - name: Git clone the repository
        uses: {% data reusables.actions.action-checkout %}
      - name: configure aws credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: arn:aws:iam::1234567890:role/example-role
          role-session-name: samplerolesession
          aws-region: {% raw %}${{ env.AWS_REGION }}{% endraw %}
      # Upload a file to AWS s3
      - name:  Copy index.html to s3
        run: |
          aws s3 cp ./index.html s3://{% raw %}${{ env.BUCKET_NAME }}{% endraw %}/
```
