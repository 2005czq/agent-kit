# agent-kit

本机 Agent 共用的 Skill 与 MCP 资产仓库，由 [Kasetto](https://github.com/pivoshenko/kasetto) (`kst`) 管理。

```
agent-kit/
├── local/                     # 本地 vendored skills
├── mcps/                      # MCP 服务器描述文件
├── skills/                    # Skill 预设，按提供方分片
├── kasetto.yaml               # 全局 kasetto 配置
└── AGENTS.md                  # 本机 Agent 共用系统提示词
```

## 使用

**初始化**：`cd <project> && printf 'scope: project\nagent: [claude-code, codex]\n' > kasetto.yaml`

**配置**：

- 项目配置编辑项目内 `kasetto.yaml`，全局配置编辑本仓库 `kasetto.yaml`。
- Skills 通过 `extends` 引用预设，MCP 直接写在同一条 `mcps:` 列表里。
- 只通过 agent-kit 里已有的 Skills 和 MCPs 安装，不在项目里直接 `kst add` 上游。
- `extends` 必须用绝对路径，`~` 不会展开。

```yaml
# <project>/kasetto.yaml
scope: project
agent: [claude-code, codex]
extends:
  - /home/zihim/agent-kit/skills/lark/all.yaml       # 安装 Lark 所有 Skills
  - /home/zihim/agent-kit/skills/stitch/design.yaml  # 安装 Stitch 设计相关 Skill
mcps:
  - source: /home/zihim/agent-kit
    mcps: [amap, stitch]                   # 安装 Stitch 和 Amap MCP
```

**部署**：项目目录下 `kst sync`；全局配置改完本仓库 `kasetto.yaml` 后也必须 `kst sync`。更新 `${kst_<name>}` 后需要 `kst sync --update`。

## 更新 agent-kit

### AGENTS.md

直接编辑保存，文件通过软链接更新到各个 Agent 的系统提示词中。

### Skills

创建或编辑预设文件 `skills/<provider>/<name>.yaml`，按提供方和类别组织。

**从上游 Git 源** (优先)：声明 `source` + `branch/ref` + `sub-dir` + `skills`。

```yaml
# skills/stitch/utilities.yaml
skills:
  - source: https://github.com/google-labs-code/stitch-skills
    branch: main                                                    # 或 ref: <tag|commit>
    sub-dir: plugins/stitch-utilities                               # skill 所在子目录 (可选)
    skills: [design-md, enhance-prompt, stitch-loop, taste-design]  # 使用 "*" 安装全部
```

**从本地**：直接指向绝对路径，本地 vendored skills 放在 `local/` 目录。

```yaml
# skills/mineru/all.yaml
skills:
  - source: /home/zihim/agent-kit/local
    skills: [mineru-document-extractor]

# skills/linuxdo/all.yaml
skills:
  - source: /home/zihim/linuxdo-helper/cli/linuxdo
    skills: [linuxdo]
```

**组合预设**：分片 identity（`source` + `ref|branch` + `sub-dir`）不同时，使用 `all.yaml` 和 `extends` 分片作为该提供方总入口。同一 identity 的多个名字必须写在**同一条** `skills:` / `mcps:` 列表里。

```yaml
# skills/stitch/all.yaml
extends:
  - ./design.yaml
  - ./build.yaml
  - ./utilities.yaml
```

### MCPs

创建或编辑 MCP 描述文件 `mcps/<name>.json`。内容必须是完整的 `mcpServers` 包裹结构。`url` 查询串和 `headers` 里的 `${kst_<name>}` 在 `sync` 时从 `~/.config/kasetto/credentials.yaml` 替换。

```json
// mcps/stitch.json
{
  "mcpServers": {
    "stitch": {
      "type": "http",
      "url": "https://stitch.googleapis.com/mcp",
      "headers": {
        "X-Goog-Api-Key": "${kst_stitch}"
      }
    }
  }
}
```
