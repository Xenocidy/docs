---
title: Configurando o OpenID Connect no Amazon Web Services
shortTitle: Configuring OpenID Connect in Amazon Web Services
intro: Use o OpenID Connect dentro de seus fluxos de trabalho para efetuar a autenticação com Amazon Web Services.
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
ms.contentlocale: pt-BR
ms.lasthandoff: 09/05/2022
ms.locfileid: '146171850'
---
{% data reusables.actions.enterprise-beta %} {% data reusables.actions.enterprise-github-hosted-runners %}

## Visão geral

O OpenID Connect (OIDC) permite que seus fluxos de trabalho de {% data variables.product.prodname_actions %} acessem os recursos na Amazon Web Services (AWS), sem precisar armazenar as credenciais AWS como segredos de {% data variables.product.prodname_dotcom %} de longa duração. 

Este guia explica como configurar a AWS para confiar no OIDC do {% data variables.product.prodname_dotcom %} como uma identidade federada e inclui um exemplo de fluxo de trabalho para o [`aws-actions/configure-aws-credentials`](https://github.com/aws-actions/configure-aws-credentials) que usa tokens para autenticação na AWS e acesso aos recursos.

## Pré-requisitos

{% data reusables.actions.oidc-link-to-intro %}

{% data reusables.actions.oidc-security-notice %}

## Adicionando o provedor de identidade ao AWS

Para adicionar o provedor OIDC do {% data variables.product.prodname_dotcom %} ao IAM, confira a [documentação da AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html).

- Para a URL do provedor: use {% ifversion ghes %}`https://HOSTNAME/_services/token`{% else %}`https://token.actions.githubusercontent.com`{% endif %}
- Em "Público-alvo": use `sts.amazonaws.com` se você estiver usando a [ação oficial](https://github.com/aws-actions/configure-aws-credentials).

### Configurando a função e a política de confiança

Para configurar a função e a relação de confiança no IAM, confira ["Como assumir uma função"](https://github.com/aws-actions/configure-aws-credentials#assuming-a-role) e ["Como criar uma função para identidade da Web ou federação do OpenID Connect"](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html) na documentação da AWS.

Edite o relacionamento de confiança para adicionar o campo `sub` às condições de validação. Por exemplo:

```json{:copy}
"Condition": {
  "StringEquals": {
    "{% ifversion ghes %}HOSTNAME/_services/token{% else %}token.actions.githubusercontent.com{% endif %}:aud": "sts.amazonaws.com",
    "{% ifversion ghes %}HOSTNAME/_services/token{% else %}token.actions.githubusercontent.com{% endif %}:sub": "repo:octo-org/octo-repo:ref:refs/heads/octo-branch"
  }
}
```

## Atualizar o seu fluxo de trabalho de {% data variables.product.prodname_actions %}

Para atualizar seus fluxos de trabalho para o OIDC, você deverá fazer duas alterações no seu YAML:
1. Adicionar configurações de permissões para o token.
2. Use a ação [`aws-actions/configure-aws-credentials`](https://github.com/aws-actions/configure-aws-credentials) para trocar o token OIDC (JWT) por um token de acesso à nuvem.

### Adicionando configurações de permissões

 {% data reusables.actions.oidc-permissions-token %}

### Solicitando o token de acesso

A ação `aws-actions/configure-aws-credentials` recebe um JWT do provedor OIDC do {% data variables.product.prodname_dotcom %} e solicita um token de acesso da sua instância da AWS. Para obter mais informações, confira a [documentação](https://github.com/aws-actions/configure-aws-credentials) da AWS.

- `<example-bucket-name>`: adicione o nome do bucket S3 aqui.
- `<role-to-assume>`: substitua o exemplo pela função da AWS.
- `<example-aws-region>`: adicione o nome da região da AWS aqui.

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
