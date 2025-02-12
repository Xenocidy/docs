---
ms.openlocfilehash: 59e70683dad451b603d2d34286976bfaa8d1d9c8
ms.sourcegitcommit: fcf3546b7cc208155fb8acdf68b81be28afc3d2d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/10/2022
ms.locfileid: "145065816"
---
任何人都可以复刻公共仓库，然后提交建议更改仓库 {% data variables.product.prodname_actions %} 工作流程的拉取请求。 虽然来自复刻的工作流程无法访问敏感数据（如密钥），但如果出于滥用目的进行修改，可能会让维护者感到烦恼。

为了帮助防止这种情况，某些外部贡献者向公共仓库提出的关于拉取请求的工作流程不会自动运行，可能需要先批准。 默认情况下，所有首次贡献者都需要批准才能运行工作流程。

{% note %}

注意：`pull_request_target` 事件触发的工作流在基础分支的上下文中运行。 由于基础分支被视为是受信任的，因此由这些事件触发的工作流将始终运行，无论审批设置如何。

{% endnote %}
