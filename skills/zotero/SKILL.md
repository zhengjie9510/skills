---
name: zotero
description: 当用户需要在本机 Zotero 个人库中搜索、阅读、整理或修改文献时使用。通过 curl 调用 Local API，查询尽量少返回字段；写入按官方文档操作。
license: Apache-2.0
---

# Zotero Local API（个人库）

使用 `curl` 访问本机 Zotero，**只操作个人库**，不需要 Python 脚本。

- 地址：`http://localhost:23119/api/users/0`（`0` 表示当前用户）。
- Zotero 必须正在运行，并在「设置 → 高级」开启「允许此计算机上的其他应用程序与 Zotero 通信」。
- **无需事先测试连接**：直接请求所需接口；如连接失败，提示检查 Zotero 是否运行及本地 API 是否开启，不编造结果。

## 查询：只拿需要的信息

1. **先筛选，再读取**：按关键词 `q`、标签 `tag`、类型 `itemType` 查找；默认 `limit=10`，更多结果再用 `start` 分页。
2. **默认只输出 ID 和标题**：用 `jq`（需已安装）过滤响应。需要摘要才加入 `abstractNote`，其他字段也按需添加；不要把完整 JSON、全文或大段笔记直接塞进上下文。
3. 只有用户任务确实需要时，才继续读取附件、笔记或全文。`jq` 减少的是给 AI 看的输出，不是 Zotero 返回的数据量。

下面的路径均接在 `http://localhost:23119/api/users/0` 后；`{key}` 是条目、分类或搜索的 ID。

| 用途 | GET 路径 |
|---|---|
| 顶层文献 / 全部条目 | `/items/top` / `/items` |
| 单条文献 / 它的附件与笔记 | `/items/{key}` / `/items/{key}/children` |
| 分类 / 顶层分类 / 子分类 | `/collections` / `/collections/top` / `/collections/{key}/collections` |
| 分类中的文献 | `/collections/{key}/items/top` |
| 标签 / 单条文献的标签 | `/tags` / `/items/{key}/tags` |
| 已保存搜索 / 搜索结果 | `/searches` / `/searches/{key}/items` |
| 回收站 | `/items/trash` |
| 附件全文索引 / 本地附件地址 | `/items/{key}/fulltext` / `/items/{key}/file/view/url` |

其他常用参数：`sort`（排序字段）、`direction`（升降序）、`qmode=everything`（搜索已索引内容）、`format=keys`（只返回 ID）。中文和空格使用 `--data-urlencode`。

**示例：搜索 Transformer，只返回 ID 和标题**

```bash
curl --noproxy '*' -fsSG \
  --data-urlencode 'q=Transformer' --data-urlencode 'limit=10' \
  'http://localhost:23119/api/users/0/items/top' \
  | jq -r '.[] | [.key, (.data.title // "")] | @tsv'
```

如果要标题和摘要，把 `jq` 过滤表达式换成：

```jq
[.[] | {key, title: (.data.title // ""), abstract: (.data.abstractNote // "")}]
```

## 写入：用官方文档，别重复造教程

Zotero 10+ 支持新增、修改、删除个人库内容：

| 对象 | 常用方法与路径 |
|---|---|
| 文献、笔记 | `POST /items`、`PATCH /items/{key}`、`PUT /items/{key}`、`DELETE /items/{key}` |
| 分类 | `POST /collections`、`PATCH /collections/{key}`、`DELETE /collections/{key}` |
| 已保存的搜索 | `POST /searches`、`PATCH /searches/{key}`、`DELETE /searches/{key}` |
| 其他 | `DELETE /tags?tag=...`、`PUT /items/{key}/fulltext`、`POST /items/{key}/file`（附件上传） |

**默认只读。** 只有用户明确要求变更才写入；删除或批量改动先确认。执行前按下方文档核对请求体、`Zotero-Server-ID`、本地写入授权（`POST /api/local/authorize`）、版本条件和响应。优先用 `PATCH` 修改指定字段；注意标签、分类等数组字段可能被整体替换。不要泄露本地授权密钥。遇到版本冲突先重新读取，不直接覆盖。

## 官方文档（需要细节时直接查）

- [Local API：本地连接、授权与差异](https://www.zotero.org/support/dev/web_api/v3/local_api)
- [Basics：完整读取接口和查询参数](https://www.zotero.org/support/dev/web_api/v3/basics)
- [Write Requests：新增、修改、删除的请求格式与版本控制](https://www.zotero.org/support/dev/web_api/v3/write_requests)
- [File Uploads：附件上传流程](https://www.zotero.org/support/dev/web_api/v3/file_upload)
- [Full-Text Content：全文读取与写入](https://www.zotero.org/support/dev/web_api/v3/fulltext_content)

文档里 Web API 的 `/users/{userID}` 路径，在本机个人库改用 `/api/users/0`；写入授权以 **Local API 文档**为准，不要使用云端 API Key。
