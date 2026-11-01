# Readar MCP

在 **WorkBuddy、Codex、Claude Code** 中连接自己的 Readar 账号，搜索收藏的文章、播客、问答和笔记，也可以把文字或链接保存到 Readar 收集箱。

Readar 提供托管的远程 MCP，使用 **Streamable HTTP + OAuth**。接入远程 MCP 不需要安装本仓库的 Node.js CLI，也不需要自行部署服务端。每位用户用自己的 Readar 账号授权。

## 一句话接入

把下面这句话直接发给 WorkBuddy、Codex 或 Claude Code：

> 请阅读 https://github.com/jerrylee-1992/readar-mcp/blob/main/README.md ，按照当前客户端的说明把 Readar 远程 MCP 添加到用户级配置，保留已有连接器，引导我在浏览器中完成 Readar 账号授权，最后查询最近 5 条笔记验证连接。

这句话用于配置客户端；邮箱验证码和授权确认仍由你在浏览器中完成。若客户端无法读取 GitHub，也可以直接使用下方配置。

## MCP 服务信息

| 项目 | 值 |
| --- | --- |
| 服务名称 | `readar` |
| 服务地址 | `https://readar-api.starmind.tech/mcp` |
| 传输方式 | Streamable HTTP |
| 认证方式 | OAuth，使用自己的 Readar 账号 |

以下配置只添加 Readar；如果已经有其他 MCP，保留它们，只合并 `readar` 对应条目。OAuth 凭据由客户端管理，不在配置里填写固定 Token、API Key 或 `client_secret`。

## WorkBuddy

1. 打开 **「专家·技能·连接器」→「连接器」→「自定义连接器」**，进入 MCP JSON 配置入口，添加下方内容：

   ```json
   {
     "mcpServers": {
       "readar": {
         "type": "streamableHttp",
         "url": "https://readar-api.starmind.tech/mcp",
         "timeout": 30000
       }
     }
   }
   ```

2. 保存，在列表中找到 `readar`，按界面提示信任、启用并发起连接。
3. 浏览器打开 Readar 授权页后，按下方「账号授权」完成登录；如果浏览器询问是否打开 WorkBuddy，允许返回客户端。
4. 在对话中启用 Readar，并按下方示例验证工具调用。

使用标准服务端 OAuth，省略 `auth_mode` 和固定 `Authorization` 请求头。服务端覆盖了 WorkBuddy 的私有协议和本机回环回调，但 **WorkBuddy 桌面端完整授权及工具调用仍待实际验收**；配置说明不等于客户端已经连通。

参考：[WorkBuddy 连接器操作指南](https://www.workbuddy.cn/docs/workbuddy/From-Beginner-to-Expert-Guide/Function-Description/Connector)、[WorkBuddy OAuth 说明](https://open.workbuddy.cn/docs/connector)。

## Codex

在终端运行：

```sh
codex mcp add readar --url https://readar-api.starmind.tech/mcp
codex mcp login readar
```

第一条添加服务，第二条发起浏览器 OAuth 授权。完成后，用 `codex mcp list` 检查配置，并重新连接 MCP 或打开新会话后验证工具调用。

也可以在 `~/.codex/config.toml` 中合并以下配置，再执行 `codex mcp login readar`：

```toml
[mcp_servers.readar]
url = "https://readar-api.starmind.tech/mcp"
```

参考：[OpenAI 官方 MCP 文档](https://developers.openai.com/codex/mcp/)。

## Claude Code

在终端运行以下命令，把 Readar 添加到所有项目可用的用户级配置：

```sh
claude mcp add --transport http --scope user readar https://readar-api.starmind.tech/mcp
```

然后进入 Claude Code，执行 `/mcp`，选择 `readar` 并按提示发起认证。浏览器授权完成后返回会话，再通过 `/mcp` 检查状态并验证工具调用。

参考：[Claude Code 官方 MCP 文档](https://code.claude.com/docs/en/mcp)。

## 账号授权

1. 在客户端发起连接或 OAuth 登录，打开 Readar 授权页。
2. 输入**已绑定 Readar 账号的邮箱**，收取并在 Readar 页面填写验证码；只有 Apple 登录的账号，先在 Readar App 中绑定邮箱。
3. 核对当前账号和客户端，选择需要的权限，点击「允许连接」，然后返回客户端。

| 权限 | 用途 |
| --- | --- |
| `readar:read` | 查询和读取自己的收藏、播客、问答及笔记 |
| `readar:collect` | 向自己的收集箱保存文字和链接 |

只需要查询时，可以不授予收藏权限。需要换账号时，先断开客户端连接，重新授权，并在 Readar 授权页退出旧账号后登录。

## 验证连接

授权完成后，先发送只读请求：

> 使用 Readar 列出我最近的 5 条笔记。

确认客户端实际调用了 Readar 工具，返回你的笔记或明确的空结果。还可以尝试：

> 在我的 Readar 中搜索「人工智能」相关的文章和播客，列出来源。

已授予收藏权限时，再按需发送：

> 把这段文字保存到 Readar 收集箱，标题是「MCP 接入测试」：这是一条测试记录。

连接成功以实际完成账号授权和工具调用为准。未授权请求返回 `401` 是正常认证挑战；从客户端发起 OAuth 登录即可。授权过期时，使用客户端的重新连接或登录入口。

## CLI（可选兼容入口）

本仓库原名 `readar-cli`，同时保留独立的 TypeScript / Node.js CLI 与 [CLI Skill](skills/readar/SKILL.md)。下面的安装方式用于 CLI；CLI 包名仍为 `readar-cli`，可执行命令仍为 `readar`，没有发布名为 `readar-mcp` 的 npm 包。

A standalone TypeScript / Node.js client for agents to save AI output into Readar and search the signed-in user's saved articles, podcast transcripts, questions/answers and personal thoughts.

Requires Node.js 22 or newer. The CLI talks only to the Readar HTTP API. It has no backend, database or AI-provider dependencies.

### One-sentence installation for agents

Tell your agent:

> Please read https://github.com/jerrylee-1992/readar-mcp/blob/codex/agent-install/skills/readar/SKILL.md, install the CLI and guide me through Readar login.

The [Readar Skill](skills/readar/SKILL.md) covers installation, email login, collection, unified semantic search, question/note listings, pagination and error handling. It is self-contained and included in the compiled package. Agents can read it directly; agents with native Skill support can install the `skills/readar` folder using their supported skill installer. Reading the URL alone does not register a skill for future automatic discovery.

With Node.js 22+, npm and Git installed, install into npm's execution cache and run directly from this public repository with one command:

```sh
npx -y --package='git+https://github.com/jerrylee-1992/readar-mcp.git#codex/agent-install' readar --help
```

The Git installation builds the executable automatically. No npm registry release, GitHub login or administrator permissions are required. The ref selects the installation-enabled branch; it can be replaced with a reviewed commit SHA for reproducible installs.

Use the same prefix for subsequent commands, for example:

```sh
npx -y --package='git+https://github.com/jerrylee-1992/readar-mcp.git#codex/agent-install' readar auth status
```

After verifying installation, the agent checks login status and guides you through email verification if needed. The default API is `https://readar-api.starmind.tech`; no API URL is needed for the official service. The agent guide also covers a permanent standalone executable installed from source.

### Install from a checkout

```sh
npm ci
npm run build
npm install -g .
readar --help
```

To run without global installation:

```sh
node dist/main.js --help
```

This project has **not** been published to npm. Use the GitHub URL above; do not install a similarly named registry package. `npm pack` builds the compiled package automatically.

### Sign in

The CLI defaults to `https://readar-api.starmind.tech`. Start email login without configuring an API URL:

```sh
readar auth login --email you@example.com
```

In a terminal, this sends the verification email and prompts for the code. In a noninteractive agent process, it returns a JSON challenge without prompting. Complete the login in a second command:

```sh
readar auth verify --code 123456
readar auth status
```

`auth verify --challenge-id <id> --code <code>` also supports an explicitly supplied challenge ID.

The API URL is remembered when a code is sent, and after successful verification. Any command can override it using `--api-url` or `READAR_API_URL`; precedence is flag, environment, saved URL, then the default API. Existing configured deployments are preserved. Custom URLs must use HTTPS; HTTP is accepted only on localhost.

The backend already supports email login and email binding. Users with Apple-only accounts must bind an email in the Readar App first. The CLI does not implement Apple login or App device authorization. The backend's account access policy still applies; using an unbound email may create a different account or be denied by that policy.

Access and refresh tokens are saved in per-API files with permissions `0600` under `${XDG_CONFIG_HOME:-~/.config}/readar`, or `READAR_CONFIG_DIR`. They are never printed. Tokens are automatically refreshed, and concurrent CLI processes share a session lock so a rotating refresh token is used once. The backend currently expires sessions after 30 days; reauthentication may be required sooner if a session is revoked. Login uses an independent session, leaving App sessions intact.

```sh
readar auth logout          # Revoke this session and remove local credentials
readar auth logout --local  # Remove local credentials without contacting the API
```

A failed remote logout keeps credentials so you can retry. If a process is forcibly killed while holding a lock, subsequent commands return `session_busy` after 35 seconds. Confirm no Readar CLI process is running, then remove the corresponding `session-*.json.lock` file from the config directory. Never remove a lock held by a live process.

### Save AI output to the collector

Choose exactly one input method:

```sh
readar collect --file answer.md --note "AI research output"
cat answer.md | readar collect --stdin
readar collect --text "A generated explanation"
readar collect --url https://example.com/article
```

Results include `article.id`, `article.status`, and `created`. Plain text is stored as ready content. URLs use the existing ingestion flow. The backend deduplicates identical trimmed text or canonical URLs for the signed-in user; a duplicate returns the existing article with `created: false`, preserving its existing metadata and archive state. For Markdown, the first line becomes the collector title. Markdown is stored as submitted text; it is not converted to HTML.

### Search saved content semantically

Describe the idea you want to find; the default hybrid mode combines keyword and semantic retrieval:

```sh
readar search --query "让 AI 帮我操作软件" --type all
readar search --query "浏览器代理" --type articles --exclude-archived
readar search --query "怎样记住读过的内容" --type episodes
readar search --query "怎样记住读过的内容" --type questions
readar search --query "读完后用自己的话复述" --type thoughts
readar search --query "Agent" --mode keyword --limit 20
```

`--query` is required and accepts 1–500 characters after trimming. Choose `--type all|articles|episodes|questions|thoughts` (default `all`). `all` includes all four content types; `episodes` searches podcast titles and generated transcripts, `questions` searches question text, full answers and answer summaries, and `thoughts` searches only notes whose kind is `thought`, excluding question-kind notes. The scope includes only the signed-in account's saved content. Archived articles are included by default; `--exclude-archived` excludes them and has no effect on the other types. Title and body are always searched together; there is no `--fields` option. Existing `questions list` and `notes list` commands below retain keyword, date and episode filters.

`--mode hybrid|keyword` defaults to `hybrid`. Hybrid requires both retrieval paths to succeed. The CLI never silently downgrades to keyword mode or scans local records. Use `--mode keyword` explicitly when you want keyword retrieval. `--limit` accepts 1–50, default 20. Each page makes one authenticated `GET /v1/search` request, apart from normal session refresh if needed.

JSON results preserve each record's `id`, `type`, `title`, `source_version`, original `snippets` and `anchor`, plus `search_mode`, `consistency`, `indexing_status`, `candidate_limit_reached`, `has_more` and `next_cursor`. Article `paragraph_index` is zero-based; podcast `segment_number` identifies a transcript segment. Episode transcript playback positions require the corresponding `track_version`. For questions/thoughts, `paragraph_index` locates the text chunk, `position_seconds` is the record's canonical episode position or null, and `segment_number`/`track_version` are null. That canonical position does not establish a natural/generated transcript track seek time. A null anchor or time must stay null; do not estimate playback time from text length. Snippets are source excerpts, not generated summaries.

Question/thought hits also preserve `record_id`, `episode_id` and `episode_title`. Their `id` is an opaque composite search identity; do not split it or pass it to a detail command. Retrieve the exact parent-scoped record using its raw `record_id` and `episode_id`:

```sh
readar questions get <record_id> --episode <episode_id>
readar notes get <record_id> --episode <episode_id>
```

Use `questions get` for a `questions` hit and `notes get` for a `thoughts` hit.

```sh
readar search --query "怎样记住读过的内容" --type episodes --cursor '<next_cursor>'
```

Treat the returned cursor as opaque and pass it unchanged with the same API, account, query, type, archive selection and mode. The page limit may change. Unified search cursors and question/note listing cursors cannot be exchanged; a `questions` search cursor cannot continue a `thoughts` or `all` search. Continue until `next_cursor` is null, including after a page with no items. Content changes and permission checks can remove candidates from an intermediate page. `candidate_limit_reached: true` means the bounded candidate set reached its cap; it is not an exhaustive count of matches. Narrow the query or content type when more coverage is needed.

`consistency: "eventual"` means newly saved or changed content can take time to appear. `indexing_status: "partial"` reports incomplete index coverage; `ready` reports the backend's current coverage status. Neither an empty page nor a partial index proves the account has no matching content. The server rechecks source versions and ownership before returning evidence.

**Availability:** Use a reviewed CLI revision whose `--help` lists `search` and the required `questions`/`thoughts` scopes; the installation branch above may predate these features. The selected backend must separately deploy, configure and backfill `GET /v1/search` for all required types. Installing this package or its skill does not enable semantic search on a server. An unconfigured or unavailable backend returns HTTP 503 `search_unavailable` (exit 1), with no successful empty fallback. HTTP 410 `search_expired` (exit 2) means start a new search without `--cursor`; HTTP 422 `invalid_cursor` means discard the old cursor and restart. This repository's loopback tests do not establish production deployment or index readiness.

### List questions and notes by keyword or filters

```sh
readar questions list --query "asset allocation" --limit 20
readar questions list --episode <episode-id>
readar questions get <question-id>
readar notes list --query "idea" --kind thought --since 2026-10-01
readar notes get <note-id> --episode <episode-id>
```

The existing `questions list --query` covers question text, answers and answer summaries; heard-answer text is a subset of the generated answer. `notes list --query` covers note text. These listing queries use the server’s Tablestore full-text index with Chinese tokenization and case-insensitive English matching; use unified `search --type questions|thoughts` above for semantic retrieval. A listing query requires matching all analyzed words within at least one searched field. Newly saved or edited content may take seconds to tens of seconds to appear. Notes include both `thought` and `question` kinds unless filtered. Queries use the signed-in user's existing API permissions.

Every returned row includes `episode_id` and `episode_title`. Question responses preserve full answers, status, source article IDs and available listening information. Notes preserve text, position, revision and timestamps.

`--since` is inclusive and `--until` is exclusive. Dates (`YYYY-MM-DD`) mean UTC midnight; timestamps require an explicit timezone. To include the whole day of October 5, use `--until 2026-10-06`.

Lists without `--query` are ordered by creation time descending, then record ID and episode ID descending. Full-text queries are ordered by relevance. Use the returned `next_cursor` unchanged with the same filters and API:

```sh
readar questions list --limit 20 --cursor '<next_cursor>'
```

`--limit` accepts 1–100, default 30. A different limit may be used for the next page. Cursors are bound to the signed-in account; changing accounts invalidates them. An account change during a command aborts with `account_changed` instead of mixing results. For database listings, records created after the previous page's boundary appear in a fresh listing, not subsequent pages.

**Server requirement:** Deploy the Readar global endpoints `GET /v1/questions`, `GET /v1/notes`, and their `/{id}` detail routes. Each CLI listing page makes one authenticated API request, including with `--episode`. There is no episode traversal or automatic scan fallback. `search_mode` is `tablestore` for a full-text query and `server_list` for an unfiltered listing. `consistency` reports `eventual` or `database` respectively.

Search requires server-side configuration and history backfill. An unavailable or unconfigured search index produces `search_unavailable` (exit 1), never an empty successful response. The CLI holds only Readar login credentials; Tablestore credentials stay on the server. Deletions and account ownership are checked against the source database before a candidate is returned, so an index waiting to synchronize cannot expose deleted records. An updated record may temporarily match an old keyword while returning the current source content.

Cursors expire after one hour, are bound to the user, API and filters, and may be reused with a different page limit. Old `client_scan` cursors must be discarded and the listing restarted. Changes during pagination may shift search results; queries are not transactional snapshots. A page can have no items and still carry `next_cursor` when index candidates were deleted, so agents should continue until `next_cursor` is null.

### Agent output contract

Success: one JSON value on stdout. Failure: one JSON object on stderr, with empty stdout. Interactive prompts also go to stderr. `--help` and `--version` return text. Remote error response messages are not echoed, to avoid leaking request data from upstream proxies.

```json
{"error":{"code":"login_required","message":"Run readar auth login to sign in."}}
```

| Exit | Meaning |
|---|---|
| 0 | Success |
| 1 | Network, configuration, server error, rate limit or busy session |
| 2 | Invalid command, input, verification code or API validation failure |
| 3 | Missing, expired or revoked login |
| 4 | Record not found |

API requests time out after 15 seconds and do not follow redirects. Collection and refresh requests are not retried after ambiguous network failures. Collection can be explicitly resubmitted: the backend's content deduplication prevents duplicate entries. A failed refresh after server-side rotation may require login again.

See [agent installation instructions](docs/agent-install.md) for the intended one-sentence onboarding flow.

### Develop and package

```sh
npm ci
npm run check
npm test
npm pack
```

The tests execute the built CLI as independent processes against real loopback HTTP servers. They cover the API contracts, content-search scopes and modes, source anchors, partial/empty pages, cursor isolation and expiry, pagination, filtering, authentication, refresh rotation, concurrent processes, input validation and credential redaction. They do not prove production mail delivery, a production account login, semantic retrieval quality or backend deployment.

The package includes compiled JavaScript and documentation. TypeScript and Node type definitions are development-only dependencies. `npm pack` builds automatically; registry publishing is a separate maintainer action.
