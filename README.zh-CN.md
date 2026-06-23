[English](README.md) · __简体中文__

---

# Calibre 新闻投递

![](banner.png)

利用 GitHub Actions 让 Calibre 定时把新闻/期刊转成电子书，通过邮箱投递到 Kindle、iPad 等设备。

## 快捷入口

 __[上传 Recipe](../../upload/master)__ | __[日更清单](../../edit/master/recipe_daily.txt)__ | __[周更清单](../../edit/master/recipe_weekly.txt)__ | __[日更工作流](../../actions/workflows/daily.yml)__ | __[周更工作流](../../actions/workflows/weekly.yml)__ | __[环境变量](../../settings/environments)__ | [开启/关闭](../../settings/actions) | [删除](../../settings#danger-zone)

## 本仓库概览

本仓库按更新频率拆分投递，并把每类内容送到最适合阅读它的设备：

| 频率 | 内容源 | Recipe 清单 | 投递目标 |
|---|---|---|---|
| __日更__ | 卫报 The Guardian（自定义 RSS recipe，仅当天新闻） | `recipe_daily.txt` | `TO_NEWS` —— 普通邮箱，iPad/手机上用 Apple Books（图书）阅读 |
| __周更__（周一） | 纽约书评 NYRB、The Diplomat、Nautilus、经济学人 The Economist（自定义 RSS recipe） | `recipe_weekly.txt` | `TO` —— Kindle（`@kindle.com`） |

报纸走普通邮箱 + Apple Books，是为了绕开「Send to Kindle」服务端转换在大体积刊物上常见的 **E999** 报错。经济学人是免费尽力版（正文多为付费墙，免费板块之外通常只有标题 + 摘要）。

## 使用说明

1) 用右上角的 __[use this template]__ 按钮创建你自己的项目。
2) 进入 [ [设置](../../settings) > __[环境变量 Environments](../../settings/environments)__ ]。
3) 点击 "__New environment__" 新建一个名为 `calibre-news` 的环境。
4) 给该环境添加如下 "__environment secrets__"：

|名称|必填|说明|示例|
|---|---|---|---|
|FROM|是|你的发件邮箱|xxx@gmail.com|
|TO|是|周更（期刊）的投递目标|xxx@kindle.com|
|TO_NEWS|否|日更（新闻）的投递目标；不填则回退到 TO|xxx@gmail.com|
|ENCRYPT|是|SMTP 加密方式|SSL|
|SECRET|是|SMTP 密码|xxxxxxxxxx|
|SMTP|是|SMTP 服务器|smtp.gmail.com|
|PORT|是|SMTP 端口|465|
|FORMAT|否|电子书格式（默认 epub）|epub|
|SIZE|否|附件大小上限（默认 25MB）|25|
|DAYS|否|电子书在 Artifacts 的保留天数（默认 90 天）|90|

> ⚠️ 若要把日更新闻送到 iPad，请务必填 `TO_NEWS`（你能在 iPad 上收信的普通邮箱）；不填会回退到 Kindle 地址，报纸又会触发 E999。

5) 进入 "__[Actions](../../actions)__"，运行 __Daily Delivery__（或 __Weekly Delivery__）的 __Run workflow__ 测试。邮件到 iPad 后，点开附件 →「拷贝到图书」即可在 Apple Books 阅读。

## 定时

投递拆成两个相互独立的工作流，各有自己的 cron 和 recipe 清单：

| 工作流 | 默认 cron | Recipe 清单 | 投递目标 |
|---|---|---|---|
| __[Daily Delivery](../../actions/workflows/daily.yml)__（`/.github/workflows/daily.yml`） | `0 0 * * *`（每天 00:00 UTC） | `recipe_daily.txt` | `TO_NEWS` |
| __[Weekly Delivery](../../actions/workflows/weekly.yml)__（`/.github/workflows/weekly.yml`） | `0 0 * * 1`（每周一 00:00 UTC） | `recipe_weekly.txt` | `TO` |

两者都调用共享的可复用工作流 `/.github/workflows/calibre-news.yml`（负责安装 Calibre、转换、发送）。该文件**没有定时、也不能手动单独运行**，只会被上面两个工作流调用，因此不会自己跑、也不会造成重复投递。

cron 时间换算参考 GitHub「__[schedule](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule)__」文档。例如你在 UTC+8、想每天早上 6:00 投递，cron 设为 `0 22 * * *`，公式为 `UTC 时间 = 本地时间 − 时区偏移`。

## Recipe

每个工作流只处理它对应清单文件（`recipe_daily.txt` 或 `recipe_weekly.txt`）里列出的 recipe，一行一个：

* __内置 recipe__：填 recipe 标题（即 recipe 文件 `Title` 属性的值），例如 `The Diplomat`。
* __自定义 recipe__：把 `.recipe` 文件放到仓库根目录，并在清单里用文件名引用，例如 `guardian-custom.recipe`。

可以为 recipe 指定封面和样式：封面图放 "__covers__" 文件夹，样式文件放 "__styles__" 文件夹，文件名需与对应的 recipe 标题或文件名一致。注意样式可能被 Send to Kindle 服务忽略。

建议在上传前先在本地测试 recipe，确保无误。

## 存储

所有转换后的电子书会打包存到 GitHub Actions 的 Artifacts，可在每次运行记录的任务详情里下载，默认保留 90 天。可在 Actions 设置页的「__[Artifact and log retention](../../settings/actions#retention-header)__」修改保留时长。超过邮件附件大小上限的文件不会通过 SMTP 发送，需手动从 Artifacts 下载。

## 相关链接

* [Recipe API 文档（英文）](https://manual.calibre-ebook.com/news_recipe.html)
* [Calibre 内置 Recipe（英文）](https://github.com/kovidgoyal/calibre/tree/master/recipes)
* [如何编写 Recipe（英文）](https://manual.calibre-ebook.com/news.html)

## 许可证

[GNU General Public License v3.0](LICENSE)
