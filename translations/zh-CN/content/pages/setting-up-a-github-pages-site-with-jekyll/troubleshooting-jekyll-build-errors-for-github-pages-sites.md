---
title: 排查 GitHub Pages 站点的 Jekyll 构建错误
intro: '您可以使用 Jekyll 构建错误消息来排查 {% data variables.product.prodname_pages %} 站点的问题。'
redirect_from:
  - /articles/page-build-failed-missing-docs-folder
  - /articles/page-build-failed-invalid-submodule
  - /articles/page-build-failed-missing-submodule
  - /articles/page-build-failed-markdown-errors
  - /articles/page-build-failed-config-file-error
  - /articles/page-build-failed-unknown-tag-error
  - /articles/page-build-failed-tag-not-properly-terminated
  - /articles/page-build-failed-tag-not-properly-closed
  - /articles/page-build-failed-file-does-not-exist-in-includes-directory
  - /articles/page-build-failed-file-is-a-symlink
  - /articles/page-build-failed-symlink-does-not-exist-within-your-sites-repository
  - /articles/page-build-failed-file-is-not-properly-utf-8-encoded
  - /articles/page-build-failed-invalid-post-date
  - /articles/page-build-failed-invalid-sass-or-scss
  - /articles/page-build-failed-invalid-highlighter-language
  - /articles/page-build-failed-relative-permalinks-configured
  - /articles/page-build-failed-syntax-error-in-for-loop
  - /articles/page-build-failed-invalid-yaml-in-data-file
  - /articles/page-build-failed-date-is-not-a-valid-datetime
  - /articles/troubleshooting-github-pages-builds
  - /articles/troubleshooting-jekyll-builds
  - /articles/troubleshooting-jekyll-build-errors-for-github-pages-sites
  - /github/working-with-github-pages/troubleshooting-jekyll-build-errors-for-github-pages-sites
product: '{% data reusables.gated-features.pages %}'
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
topics:
  - Pages
shortTitle: Troubleshoot Jekyll errors
ms.openlocfilehash: 224f626df144ad249a799767984118778202e7b4
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/05/2022
ms.locfileid: '147428330'
---
## 排查构建错误

如果在本地或 {% data variables.product.product_name %} 上构建 {% data variables.product.prodname_pages %} 站点时发生 Jekyll 错误，您可以使用错误消息排查故障。 有关错误消息以及如何查看它们的详细信息，请参阅“[关于 {% data variables.product.prodname_pages %} 站点的 Jekyll 构建错误](/articles/about-jekyll-build-errors-for-github-pages-sites)”。

如果您收到一般错误消息，请检查常见问题。
- 您使用的插件不受支持。 有关详细信息，请参阅“[关于 {% data variables.product.prodname_pages %} 和 Jekyll](/articles/about-github-pages-and-jekyll#plugins)。”{% ifversion fpt or ghec %}
- 您的仓库已超过我们的仓库大小限制。 有关详细信息，请参阅“[我的磁盘配额是多少？](/articles/what-is-my-disk-quota)”{% endif %}
- 你更改了 _config.yml 文件中的 `source` 设置。 {% ifversion pages-custom-workflow %}如果从分支发布站点，{% endif %}{% data variables.product.prodname_pages %} 在生成过程中会覆盖此设置。
- 已发布文件中的文件名包含不支持的冒号 (`:`)。

如果您收到特定的错误消息，请查看下面的错误消息疑难解答信息。

{% ifversion pages-custom-workflow %}修复任何错误后，通过将更改推送到站点的源分支（如果从分支发布）或通过触发自定义 {% data variables.product.prodname_actions %} 工作流（如果使用 {% data variables.product.prodname_actions %} 发布）来触发另一个生成。{% else %}修复任何错误后，将更改推送到站点的发布源，以在 {% data variables.product.product_name %} 上触发另一个生成。{% endif %}

## Config 文件错误

此错误意味着站点构建失败，因为 _config.yml 文件包含语法错误。

若要排除故障，请确保 _config.yml 文件遵循以下规则：

{% data reusables.pages.yaml-rules %}

{% data reusables.pages.yaml-linter %}

## 日期不是有效的日期时间

此错误意味着站点上的某个页面包含无效的日期时间。

要排除故障，请搜索错误消息中的文件和文件布局，以调用任何与日期相关的 Liquid 过滤器。 确保传递给与日期相关的 Liquid 筛选器的任何变量在所有情况下都具有值，并且永远不会传递 `nil` 或 `""`。 有关详细信息，请参阅 Liquid 文档中的“[Liquid 筛选器](https://help.shopify.com/en/themes/liquid/filters)”。

## 文件在包含目录中不存在

此错误意味着代码引用了 _includes 目录中不存在的文件。

{% data reusables.pages.search-for-includes %} 如果你引用的任何文件不在 _includes 目录中，请将这些文件复制或移动到 _includes 目录 。

## 文件是符号链接

此错误意味着你的代码引用了站点已发布文件中不存在的符号链接文件。

{% data reusables.pages.search-for-includes %} 如果你引用的任何文件是符号链接的文件，请将这些文件复制或移动到 _includes 目录中。

## 文件未采用正确的 UTF-8 编码

此错误意味着你使用了非拉丁字符（例如 `日本語`）但没有告诉计算机预期这些符号。

若要排除故障，请通过在 _config.yml 文件中添加以下行来强制使用 UTF-8 编码：
```yaml
encoding: UTF-8
```

## 高亮插件语言无效

此错误意味着你在配置文件中指定了除 [Rouge](https://github.com/jneen/rouge) 或 [Pygments](http://pygments.org/) 之外的任何语法高亮插件。

若要排除故障，请更新 _config.yml 文件以指定 [Rouge](https://github.com/jneen/rouge) 或 [Pygments](http://pygments.org/)。 有关详细信息，请参阅“[关于 {% data variables.product.product_name %} 和 Jekyll](/articles/about-github-pages-and-jekyll#syntax-highlighting)”。

## 帖子日期无效

此错误意味着站点上的帖子在文件名或 YAML 前页中包含无效的日期。

要排除故障，请确保所有日期的 UTC 格式均为 YYYY-MM-DD HH:MM:SS， 并且都是实际日历日期。 若要指定与 UTC 偏移的时区，请使用格式 YYYY-MM-DD HH:MM:SS +/-TTTT，例如 `2014-04-18 11:30:00 +0800`。

如果你在 _config.yml 文件中指定日期格式，请确保格式正确。

## Sass 或 SCSS 无效

此错误意味着您的仓库包含内容无效的 Sass 或 SCSS 文件。

要排除故障，请查看指示 Sass 或 SCSS 无效的错误消息中包含的行号。 为防止以后出错，请在您的常用文本编辑器中安装 Sass 或 SCSS 语法检查插件。

## 子模块无效

此错误意味着您的仓库包含尚未正确初始化的子模块。

{% data reusables.pages.remove-submodule %}

如果要使用子模块，请确保在引用子模块时使用 `https://`（而不是 `http://`），并且子模块位于公共存储库中。

## 数据文件中的 YAML 无效

此错误意味着 _data 文件夹中的多个文件之一包含无效的 YAML。

若要排除故障，请确保 _data 文件夹中的 YAML 文件遵循以下规则：

{% data reusables.pages.yaml-rules %}

{% data reusables.pages.yaml-linter %}

有关 Jekyll 数据文件的详细信息，请参阅 Jekyll 文档中的“[数据文件](https://jekyllrb.com/docs/datafiles/)”。

## Markdown 错误

此错误意味着您的仓库包含 Markdown 错误。

要排除故障，请确保使用受支持的 Markdown 处理器。 有关详细信息，请参阅“[使用 Jekyll 为 {% data variables.product.prodname_pages %} 站点设置 Markdown 处理器](/articles/setting-a-markdown-processor-for-your-github-pages-site-using-jekyll)”。

然后，确认错误消息中的文件使用有效的 Markdown 语法。 有关详细信息，请参阅 Daring Fireball 上的“[Markdown：语法](https://daringfireball.net/projects/markdown/syntax)”。

## 缺少 docs 文件夹

此错误意味着你已选择分支上的 `docs` 文件夹作为发布源，但该分支上存储库的根目录中没有 `docs` 文件夹。

若要排除故障，如果 `docs` 文件夹被意外移动，请尝试将 `docs` 文件夹移回你为发布源选择的分支上的存储库根目录。 如果 `docs` 文件夹被意外删除，你可以执行以下任一操作：
- 使用 Git 还原或撤消删除。 有关详细信息，请参阅 Git 文档中的“[git-revert](https://git-scm.com/docs/git-revert.html)”。
- 在为发布源选择的分支上的存储库根目录中创建一个新的 `docs` 文件夹，并将站点的源文件添加到该文件夹中。 有关详细信息，请参阅“[创建新文件](/articles/creating-new-files)”。
- 更改发布源。 有关详细信息，请参阅“[为 {% data variables.product.prodname_pages %} 配置发布源](/articles/configuring-a-publishing-source-for-github-pages)”。

## 缺少子模块

此错误意味着您的仓库包含不存在或尚未正确初始化的子模块。

{% data reusables.pages.remove-submodule %}

如果要使用子模块，请初始化子模块。 有关详细信息，请参阅 _Pro Git_ 书中的“[Git 工具 - 子模块](https://git-scm.com/book/en/v2/Git-Tools-Submodules)”。

## 配置了相对永久链接

此错误意味着 _config.yml 文件中存在 {% data variables.product.prodname_pages %} 不支持的相对永久链接。

永久链接是引用站点上特定页面的永久 URL。 绝对永久链接以站点的根目录开头，而相对永久链接以包含引用页面的文件夹开头。 {% data variables.product.prodname_pages %} 和 Jekyll 不再支持相对永久链接。 有关永久链接的详细信息，请参阅 Jekyll 文档中的“[永久链接](https://jekyllrb.com/docs/permalinks/)”。

若要排除故障，请从 _config.yml 文件中删除 `relative_permalinks` 行，并将站点中的任何相对永久链接重新格式化为绝对永久链接。 有关详细信息，请参阅“[编辑文件](/repositories/working-with-files/managing-files/editing-files)”。

## 符号链接不存在于站点的仓库中

此错误意味着你的站点包含站点已发布文件中不存在的符号链接。 有关符号链接的详细信息，请参阅 Wikipedia 上的“[符号链接](https://en.wikipedia.org/wiki/Symbolic_link)”。

要排除故障，请确定错误消息中的文件是否用于构建站点。 如果否，或者您不希望文件成为符号链接，请删除该文件。 如果符号链接文件是生成站点的必需项，请确保符号链接引用的文件或目录存在于站点已发布文件中。 若要包含外部资产，请考虑使用 {% ifversion fpt or ghec %}`git submodule` 或{% endif %}第三方包管理器，例如 [Bower](https://bower.io/)。{% ifversion fpt or ghec %} 有关详细信息，请参阅“[将子模块用于 {% data variables.product.prodname_pages %}](/articles/using-submodules-with-github-pages)”。{% endif %}

## 'for' 循环中的语法错误

此错误意味着代码在 Liquid `for` 循环声明中包含无效语法。

若要排除故障，请确保错误消息所指文件中的所有 `for` 循环都具有正确的语法。 有关 `for` 循环的正确语法的详细信息，请参阅 Liquid 文档中的“[迭代标记](https://help.shopify.com/en/themes/liquid/tags/iteration-tags#for)”。

## 标记未正确关闭

此错误消息意味着您的代码包含未正确关闭的逻辑标记。 例如，{% raw %}`{% capture example_variable %}` 必须由 `{% endcapture %}`{% endraw %} 关闭。

要排除故障，请确保错误消息所指文件中的所有逻辑标记都正确关闭。 有关详细信息，请参阅 Liquid 文档中的“[Liquid 标记](https://help.shopify.com/en/themes/liquid/tags)”。

## 标记未正确终止

此错误意味着您的代码包含未正确终止的输出标记。 例如，是 {% raw %}`{{ page.title }` 而不是 `{{ page.title }}`{% endraw %}。

若要排除故障，请确保错误消息所指文件中的所有输出标记都以 `}}` 终止。 有关详细信息，请参阅 Liquid 文档中的“[Liquid 对象](https://help.shopify.com/en/themes/liquid/objects)”。

## 未知标记错误

此错误意味着您的代码包含无法识别的 Liquid 标记。

要排除故障，请确保错误消息所指文件中的所有 Liquid 标记都与 Jekyll 的默认变量相匹配，并且标记名称没有拼写错误。 有关默认变量的列表，请参阅 Jekyll 文档中的“[变量](https://jekyllrb.com/docs/variables/)”。

不受支持的插件是无法识别标记的常见来源。 如果您通过在本地生成站点并将静态文件推送到 {% data variables.product.product_name %} 的方法在站点中使用不受支持的插件，请确保该插件未引入 Jekyll 默认变量中没有的标记。 有关受支持插件的列表，请参阅“[关于 {% data variables.product.prodname_pages %} 和 Jekyll](/articles/about-github-pages-and-jekyll#plugins)”。
