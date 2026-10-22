# CI/CD 通用教程：学会 GitHub Actions，一通百通

> **教程目标**：以 GitHub Actions 为入门载体，先建立**所有 CI/CD 工具共通的心智模型**，再逐节掌握实战语法；学完后你拿到任何陌生 CI 工具（GitLab CI、Jenkins、CircleCI、Tekton……）都能用同一套框架在 30 分钟内上手。
> **前置要求**：会用 Git、有 GitHub 账号；不需要任何 CI/CD 经验。
> **建议节奏**：第 1–3 章半天跟做；第 4 章当字典查；第 5–7 章在真实项目里对照落地。
> **配套阅读**：工具选型与定价对比见同目录 [cicd-tools-comparison-2026.md](./cicd-tools-comparison-2026.md)。

---

## 目录

1. [核心认知：五要素心智模型](#1-核心认知五要素心智模型)
2. [准备环境：10 分钟跑起第一条流水线](#2-准备环境10-分钟跑起第一条流水线)
3. [GitHub Actions 十二节渐进课](#3-github-actions-十二节渐进课)
4. [概念映射：从 GHA 翻译到其他四家](#4-概念映射从-gha-翻译到其他四家)
5. [陌生工具 30 分钟上手法（通用 Checklist）](#5-陌生工具-30-分钟上手法通用-checklist)
6. [工具无关的最佳实践](#6-工具无关的最佳实践)
7. [跨工具共通的坑](#7-跨工具共通的坑)
8. [练习路线与自测](#8-练习路线与自测)

---

## 1. 核心认知：五要素心智模型

### 1.1 一句话理解 CI/CD

**CI（持续集成）**：代码一推送，机器自动帮你"装依赖 → 检查 → 测试 → 构建"，每次提交都得到验证。
**CD（持续交付/部署）**：在 CI 之上继续自动完成"打包 → 推镜像/制品 → 发布到测试/生产环境"。

所有 CI/CD 工具——不管是 GitHub Actions 还是二十年前的 Jenkins——本质上都在回答同一个问题：

> **"当某件事发生时（事件），在什么机器上（执行器），按什么顺序（编排），跑哪些命令（步骤），并如何复用中间产物（缓存/制品）？"**

### 1.2 五要素：所有工具的最大公约数

记住下面五个词，它们是整个 CI/CD 世界的"元素周期表"：

```
 ① 触发器 Trigger    什么时候跑？（push / PR / 定时 / 手动 / API）
       │
       ▼
 ② 流水线 Pipeline   一次完整执行（workflow / pipeline / PipelineRun）
       │
       ├── ③ 任务 Job    一组步骤的容器，Job 之间可串可并
       │       │
       │       └── ④ 步骤 Step   一条命令或一个可复用动作（最小执行单元）
       │
       └── 跑在 ⑤ 执行器 Runner 上（托管虚拟机 / 容器 / 自托管机器 / K8s Pod）
```

围绕五要素，还有六个**辅助机制**（各工具都有，只是叫法不同）：

| 辅助机制 | 解决什么问题 |
|---|---|
| 环境变量 / 上下文 | 把"这次运行的信息"（分支、SHA、事件类型）传给脚本 |
| 密钥 Secrets | 安全地注入 Token/密码，不写进代码和日志 |
| 条件 Condition | 什么情况下才执行某个任务/步骤 |
| 矩阵 Matrix | 一套定义，批量跑多版本/多 OS 组合 |
| 缓存 Cache | 依赖下载一次，后续复用（省时间省钱） |
| 制品 Artifacts | Job 之间或流水线结束后传递/保存文件 |

### 1.3 一张表看懂五家工具的术语对应

**学新工具时，第一件事永远是查这张表的对应行。** 概念你已经会了，只是换了个名字。

| 通用概念 | GitHub Actions | GitLab CI | Jenkins | CircleCI | Tekton |
|---|---|---|---|---|---|
| 流水线 | workflow | pipeline | pipeline（Jenkinsfile） | pipeline | PipelineRun（Pipeline 是模板） |
| 配置文件 | `.github/workflows/*.yml` | `.gitlab-ci.yml` | `Jenkinsfile` | `.circleci/config.yml` | K8s YAML（CRD） |
| 任务 | job | job | stage | job | TaskRun / Task |
| 步骤 | step | script 里的命令 | step（sh/bat） | step（run） | step（容器） |
| 执行器 | runner | runner（executor） | agent / node | executor / runner | Pod（K8s 调度） |
| 触发器 | `on:` | `rules:` | webhook / multibranch | workflow `filters` | EventListener |
| 条件 | `if:` | `rules:` | `when` | `when:` | `when:` |
| 矩阵 | `strategy.matrix` | `parallel: matrix` | `matrix` | `matrix` + parameters | ❌ 无原生 |
| 缓存 | `actions/cache` | `cache:` | 无内建 | restore/save_cache | 无内建（PVC） |
| 制品 | artifacts | artifacts | archiveArtifacts / stash | store_artifacts | workspace / Results |
| 密钥 | secrets | masked/protected variables | credentials | contexts | K8s Secret |
| 可复用组件 | action / reusable workflow | `include:` 模板 | shared library | orb | Hub 上的 Task |
| 任务依赖 | `needs:` | `needs:` / stages | stage 顺序 | `requires:` | `runAfter:` |
| 内置环境变量 | `GITHUB_*` | `CI_*` | `BUILD_*`/`env.*` | `CIRCLE_*` | params/results |

### 1.4 为什么选 GitHub Actions 作为第一门

- **概念最直白**：YAML 里 `on → jobs → steps` 与五要素一一对应，没有历史包袱；
- **零成本起步**：公开仓库免费，私有仓库每月有免费额度；
- **生态最大**：几乎任何操作都有现成 action，学语法的同时能学会"复用"这一 CI 核心思维；
- **反馈最快**：推代码后 10 秒内就能在网页看到执行过程。

> 迁移成本提示：本节及配套报告均基于 2026-10 的现行语法与定价。工具语法相对稳定，但定价/免费额度变化频繁，涉及费用时以官方文档为准。

---

## 2. 准备环境：10 分钟跑起第一条流水线

1. 在 GitHub 新建一个仓库（公开/私有均可），本地 `git clone` 下来；
2. 在仓库根目录建一个特殊目录和文件：

```
你的仓库/
└── .github/
    └── workflows/
        └── hello.yml      ← 文件名随意，但必须放在这个目录
```

3. 写入第一个工作流：

```yaml
# .github/workflows/hello.yml
name: Hello CI                         # 工作流显示名（随便起）

on: [push]                             # ① 触发器：每次 push 都跑

jobs:                                  # ② 任务集合
  greet:                               #    任务 ID（随便起，用于互相引用）
    runs-on: ubuntu-latest             # ⑤ 执行器：GitHub 托管的最新 Ubuntu
    steps:                             # ④ 步骤列表，从上往下顺序执行
      - name: 打个招呼                #    步骤名（日志里显示）
        run: echo "Hello, CI/CD!"
      - name: 看看运行环境
        run: |
          node --version || echo "没有 node 也没关系"
          pwd
          ls -la
```

4. 提交并推送：`git add . && git commit -m "ci: first workflow" && git push`；
5. 打开 GitHub 仓库页 → **Actions** 标签页 → 点击运行记录 → 展开 `greet` 任务，就能看到每一步的实时日志。

**你已经完成了第一个 CI 闭环**：事件触发 → GitHub 分配一台全新的虚拟机 → 按 YAML 执行步骤 → 回收机器。之后所有课程都是在这个骨架上加东西。

---

## 3. GitHub Actions 十二节渐进课

> 每节都可独立跟做，建议把示例依次追加到自己的练习仓库里。

### 3.1 触发器 `on`：决定什么时候跑

```yaml
on:
  push:
    branches: [main]                   # 只在 push 到 main 时
    paths: ['src/**']                  # 只有 src/ 下文件变更才触发
  pull_request:
    branches: [main]                   # 向 main 发 PR 时
  schedule:
    - cron: '0 22 * * 1-5'            # 工作日 UTC 22:00（注意是 UTC！）
  workflow_dispatch:                   # 在 Actions 页面手动点按钮运行
    inputs:                            # 手动运行时可填参数
      env_name:
        description: '部署到哪个环境'
        required: true
        default: 'staging'
        type: choice
        options: [staging, production]
```

**心智迁移**：`on` = GitLab 的 `workflow:rules`/job `rules`、Jenkins 的 webhook 触发条件、CircleCI workflow 的 `filters`、Tekton 的 EventListener。

### 3.2 拉代码与使用现成 action：`uses`

GitHub 托管 runner 是**每次全新的虚拟机**，不会自带你的代码。绝大多数工作流第一步都是官方拉代码 action：

```yaml
steps:
  - uses: actions/checkout@v4          # @v4 是版本固定（重要，见第 7 节）
  - uses: actions/setup-node@v4        # 安装 Node.js
    with:                              # with = 给这个 action 传参数
      node-version: 20
      cache: npm                       # 顺带自动配置 npm 缓存
  - run: node -v
```

理解 `uses` 的两种来源：

- **仓库 action**：`actions/checkout@v4`（官方组织 `actions` 下）；
- **同仓库复用**：`./.github/actions/my-action`（你自己写的 composite action）。

### 3.3 上下文、变量与表达式：`${{ }}`

GitHub 提供一组**上下文对象**，用 `${{ 表达式 }}` 插值：

```yaml
jobs:
  demo:
    runs-on: ubuntu-latest
    env:                               # 任务级环境变量（对所有步骤生效）
      LOG_LEVEL: info
    steps:
      - run: echo "分支=$GITHUB_REF_NAME SHA=${GITHUB_REF##*/}"
      - run: echo "事件是 ${{ github.event_name }}，仓库是 ${{ github.repository }}"
      - name: 自定义变量
        run: echo "$GREETING"
        env:                           # 步骤级环境变量
          GREETING: "hi ${{ github.actor }}"
```

最常用的内置上下文：

| 上下文 | 典型字段 | 含义 |
|---|---|---|
| `github` | `event_name` / `ref_name` / `sha` / `actor` / `repository` / `event` | 本次运行的元数据与事件载荷 |
| `env` | 你自己定义的变量 | 环境变量 |
| `vars` | `vars.MY_VAR` | 在仓库 Settings 里配置的**非敏感**配置变量 |
| `secrets` | `secrets.NPM_TOKEN` | 加密密钥（下一节） |
| `matrix` | `matrix.node` | 当前矩阵组合的值 |
| `needs` | `needs.build.outputs.x` | 上游 job 的产物/输出 |
| `runner` | `runner.os` / `runner.temp` | 执行器信息 |

### 3.4 密钥 Secrets：注入但不泄露

在仓库 **Settings → Secrets and variables → Actions** 添加 `NPM_TOKEN`，然后：

```yaml
steps:
  - run: npm publish
    env:
      NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}   # 通过环境变量注入
```

三条铁律：

1. 密钥**不要**直接写在 `run:` 命令行参数里（会进日志和进程列表），走环境变量；
2. 日志中密钥会被自动打码，但你若故意 `echo` 成某种变形（base64 等）仍可能泄露；
3. **来自 fork 的 PR 默认拿不到密钥**——这是安全设计，不是 bug。

### 3.5 条件执行 `if`

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - run: ./deploy.sh
```

常用条件：`github.event_name == 'pull_request'`、`contains(github.event.head_commit.message, '[release]')`、`matrix` 组合、步骤级 `if: failure()` / `always()` / `cancelled()`。

### 3.6 矩阵 Matrix：一套定义，多组合执行

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false                 # 一个组合失败不要取消其他组合
      matrix:
        node: [18, 20, 22]
        os: [ubuntu-latest, windows-latest]
        exclude:                       # 排除不需要的组合
          - { os: windows-latest, node: 18 }
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: node -v
```

这会生成 (3 Node 版本 × 2 OS − 1) = **5 个并行任务**。

### 3.7 缓存 Cache：让第二次以后更快

现代 setup 系列 action 已内置语言生态缓存，优先用：

```yaml
- uses: actions/setup-node@v4
  with: { node-version: 20, cache: npm }     # 自动按 lockfile 做 key
```

通用场景用 `actions/cache`，理解 key/restore-keys 的设计是**所有工具缓存的通用原理**：

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm                        # 要缓存的目录
    key: npm-${{ runner.os }}-${{ hashFiles('package-lock.json') }}  # 精确 key
    restore-keys: |                     # 精确 key 未命中时的回退前缀
      npm-${{ runner.os }}-
```

通用模型（在 GitLab/CircleCI 完全同构）：

- **key**：缓存的身份标识，通常 = 操作系统 + lockfile 哈希，lockfile 变了就开新缓存；
- **restore-keys**：找不到精确 key 时找"最像的"旧缓存（依赖大部分没变时依然省时间）；
- **path**：缓存哪些目录。

### 3.8 制品 Artifacts：保存与传递文件

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist                    # 制品名
          path: dist/                   # 文件/目录
          retention-days: 7             # 保留天数

  deploy:
    needs: build                        # 等 build 成功才跑
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with: { name: dist }
      - run: ls -la dist
```

要点：**Job 之间不共享文件系统**（每个 job 是独立机器），跨 job 传文件只能走制品/缓存；这在所有 CI 工具里都成立。

### 3.9 多 Job 编排：串并行与 DAG

```yaml
jobs:
  lint:    { runs-on: ubuntu-latest, steps: [{ run: 'npm run lint' }] }
  test:    { runs-on: ubuntu-latest, steps: [{ run: 'npm test' }] }
  build:
    needs: [lint, test]                 # 两个都成功才构建；不写 needs 则默认并行
    runs-on: ubuntu-latest
    steps: [{ run: 'npm run build' }]
  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps: [{ run: './deploy.sh' }]
```

`needs` 构成一张 DAG（有向无环图）：lint/test 并行 → build → deploy。这正是"流水线"名称的由来。

### 3.10 并发控制：取消过时运行与防互踩

```yaml
concurrency:
  group: deploy-${{ github.ref }}       # 同一分支的运行归为一组
  cancel-in-progress: true              # 新运行来了，取消旧的（省分钟数）
```

部署类任务则相反：`cancel-in-progress: false` 防止两个部署互相打断。

### 3.11 可复用工作流：不要复制粘贴

当多个仓库/多条流水线共用一段逻辑，用 **reusable workflow**：

```yaml
# 被调用方：.github/workflows/_reusable-test.yml
on:
  workflow_call:                        # 声明"我可以被别的工作流调用"
    inputs:
      node-version: { type: number, required: true }
    secrets:
      npm-token: { required: true }
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v4
        with: { node-version: ${{ inputs.node-version }} }
      - run: npm ci && npm test
        env: { NPM_TOKEN: ${{ secrets.npm-token }} }
```

```yaml
# 调用方
jobs:
  test:
    uses: ./.github/workflows/_reusable-test.yml   # 同仓库；也可 owner/repo/.github/...@v1
    with: { node-version: 20 }
    secrets: { npm-token: ${{ secrets.NPM_TOKEN }} }
```

这是 CI 工程化的分水岭：**从"每个仓库一份 YAML"进化到"流水线即产品"**。对应 GitLab 的 `include:template`、Jenkins 的 Shared Library、CircleCI 的 orb、Tekton 的 Hub Task。

### 3.12 实战：一条完整的前端项目 CI 流水线

把前 11 节串起来：

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read                        # 最小权限原则（第 6 节详述）

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@v4
        if: always()
        with: { name: coverage-${{ matrix.os }}, path: coverage/ }

  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with: { name: dist, path: dist/, retention-days: 7 }

  deploy-staging:
    needs: build
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment: staging                # 可在 Settings 里给环境配独立密钥与保护规则
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with: { name: dist, path: dist }
      - run: ./scripts/deploy.sh staging
        env: { DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }} }
```

### 3.13（选修）自托管 Runner

不想用（或不够用）GitHub 托管机时，可以在自己的机器上装 runner：

- runner **主动向 GitHub 发出站 HTTPS 长连接**领任务，无需开放入站端口；
- 在仓库 Settings → Actions → Runners 按引导下载、执行 `./config.sh --url ... --token ...`，再 `./run.sh`；
- YAML 里把 `runs-on` 改成你打的标签即可：`runs-on: [self-hosted, linux, gpu]`；
- **安全红线**：自托管机不是一次性干净环境，不要让不可信代码（公开仓库 fork PR）在上面跑；生产用请配容器级隔离。

---

## 4. 概念映射：从 GHA 翻译到其他四家

学习策略：**不要把新工具当新东西学，只学"差异点"**。下面四张速查卡，每张都用"你已认识的 GHA 片段 → 目标工具等价片段"的方式组织。

### 4.1 GitLab CI 速查卡

整体观感：GitLab 没有文件头触发器，顶层每个键就是一个 job，`stage` 决定它在全局阶段轴上的位置。

| GHA | GitLab CI |
|---|---|
| 仓库自动 clone？❌ 需 checkout | ✅ **自动 clone**，不需要第一步 |
| `on: [pull_request]` | job 里 `rules: - if: $CI_PIPELINE_SOURCE == "merge_request_event"` |
| `runs-on: ubuntu-latest` | `image: node:20`（容器即环境）或 `tags: [gpu]` |
| `uses: actions/setup-node@v4` | `image:` 指定镜像 / `include:` 引用模板 |
| `strategy.matrix.node: [18,20]` | `parallel: matrix: [NODE_VERSION: ["18","20"]]` |
| `actions/cache` | `cache: { key: ..., paths: [...] }` |
| upload-artifact | `artifacts: { paths: [dist/], expire_in: 7 days }` |
| `needs: [a, b]` | `needs: [a, b]`（同名！但 GitLab 默认还按 stage 排序） |
| `secrets.NPM_TOKEN` | Settings → CI/CD → Variables（可设 masked/protected） |
| `${{ github.sha }}` | `$CI_COMMIT_SHA`（全套 `CI_*` 预定义变量） |
| reusable workflow | `include: - project: ... file: ... template: ...` |

GHA 片段：

```yaml
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - run: npm ci && npm test
```

GitLab 等价（`.gitlab-ci.yml`）：

```yaml
stages: [test]
test:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm test
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
```

**只需重点学的差异**：① 触发是 job 级 `rules`（同一份 YAML 里不同 job 条件可以不同）；② 默认按 `stages` 串行，想跳过阶段限制用 `needs`；③ 执行环境首选 Docker 镜像思维。

### 4.2 Jenkins 速查卡

整体观感：Jenkinsfile 是 Groovy 程序，表达力最强，也最需要工程纪律。

| GHA | Jenkins（Declarative Pipeline） |
|---|---|
| `.github/workflows/` | 仓库根 `Jenkinsfile`（Multibranch Pipeline 自动发现分支） |
| `on: push` | 由 Multibranch / webhook 插件负责，文件里不写 |
| `jobs.test` | `stage('test')` |
| `runs-on` + setup action | `agent { docker 'node:20' }` 或 `agent { label 'linux' }` |
| `- run: npm ci` | `sh 'npm ci'`（Windows 用 `bat`） |
| secrets | Jenkins 凭据库 + `withCredentials([...])` |
| matrix | `matrix { axes { axis { ... } } }` |
| 缓存 | 无内建：持久 workspace、`stash`/`unstash`（同次构建内传文件）或插件 |
| artifacts | `archiveArtifacts artifacts: 'dist/**'` |
| needs/阶段 | `stages` 顺序执行；`parallel { }` 块显式并行 |
| `post` 失败通知 | `post { failure { ... } success { ... } always { ... } }` |
| reusable workflow | Shared Library（Groovy 函数库） |
| `${{ github.sha }}` | `env.GIT_COMMIT` / `currentBuild` 等 |

等价 Jenkinsfile：

```groovy
pipeline {
  agent { docker 'node:20' }
  stages {
    stage('Test') {
      steps {
        sh 'npm ci'
        sh 'npm test'
      }
    }
  }
  post {
    always { archiveArtifacts artifacts: 'coverage/**', allowEmptyArchive: true }
    failure { echo '构建失败，去查日志吧' }
  }
}
```

**只需重点学的差异**：① 两步式（先装插件/配 agent，再写 Jenkinsfile）；② `sh` 里就是 Shell，Groovy 还能做循环/异常/动态 stage；③ 缓存与环境隔离要自己设计，这是 Jenkins 最大的运维坑。

### 4.3 CircleCI 速查卡

整体观感：**job 定义"怎么做"与 workflow 定义"何时做/做哪些"严格分离**，参数化能力贯穿始终。

| GHA | CircleCI |
|---|---|
| `on: push branches: [main]` | workflow 里 `filters: { branches: { only: [main] } }` |
| `runs-on: ubuntu-latest` | `docker: - image: cimg/node:20` 或 `machine: true` |
| checkout action | `- checkout`（内置 step） |
| matrix | job 声明 `parameters`，workflow 用 `matrix: { parameters: {...} }` |
| cache | `- restore_cache` + `- save_cache`，key 支持 `{{ checksum "package-lock.json" }}` |
| artifacts | `- store_artifacts: { path: dist }` |
| needs | workflow 里 `- deploy: { requires: [build] }` |
| secrets | Project 环境变量 / **Contexts**（一组可跨项目复用的密钥） |
| reusable | orbs（`orbs: { node: circleci/node@5 }` 然后 `node/install-packages`） |
| `${{ github.sha }}` | `$CIRCLE_SHA1` / `$CIRCLE_BRANCH` |
| 手动触发 | `workflow_dispatch` ↔ API 触发或 hold（`type: approval` 人工审批门） |

等价配置：

```yaml
version: 2.1
jobs:
  test:
    docker: [{ image: cimg/node:20 }]
    steps:
      - checkout
      - restore_cache: { keys: ['npm-{{ checksum "package-lock.json" }}'] }
      - run: npm ci
      - save_cache: { key: 'npm-{{ checksum "package-lock.json" }}', paths: [~/.npm] }
      - run: npm test
workflows:
  ci:
    jobs:
      - test:
          filters: { branches: { only: [main, /pull\/.*/] } }
```

**只需重点学的差异**：① jobs/workflows 两段式；② 缓存 key 的 `checksum` 内容寻址是最精确的模型，值得把这个思路带回任何工具；③ credits 按量计费，macOS/GPU 单价高，矩阵要算成本。

### 4.4 Tekton 速查卡

整体观感：没有"CI 服务器"，一切都是 Kubernetes 资源——你不是在写脚本编排，而是在**声明 K8s 对象**。

| GHA | Tekton |
|---|---|
| workflow YAML | `Pipeline`（模板）+ `PipelineRun`（一次执行） |
| job / step | `Task`（模板）/ Task 里的 `steps`（每个 step 一个容器） |
| `on:` | `EventListener` + `TriggerTemplate`（Webhook → 自动生成 PipelineRun） |
| needs | Pipeline tasks 里的 `runAfter: [a]` |
| matrix | ❌ 无原生，用外部循环（脚本/CI 系统）生成多个 PipelineRun |
| 跨 step 共享文件 | `workspaces`（本质 PVC 挂载） |
| cache | 无内建：PVC 预热卷 / 镜像 registry / 自建对象存储 |
| artifacts | `workspaces` + `results`（键值短文本） |
| secrets | K8s `Secret`，step 里 `env: valueFrom: secretKeyRef` 或 volume 挂载 |
| reusable action | Tekton Hub 上的 Task（`tkn hub install task git-clone`） |
| runner | Pod——K8s 调度器就是 runner 管理器 |

等价 Task（配合 Hub 的 `git-clone` Task 使用）：

```yaml
apiVersion: tekton.dev/v1
kind: Task
metadata: { name: node-test }
spec:
  params:
    - { name: node-version, type: string, default: "20" }
  workspaces:
    - { name: source }
  steps:
    - name: test
      image: node:$(params.node-version)
      workingDir: $(workspaces.source.path)
      script: |
        npm ci
        npm test
```

**只需重点学的差异**：① 前置条件是"团队已经会 Kubernetes"，否则先学 K8s 再碰 Tekton；② 触发器、缓存、通知全是独立组件，自由度高但组装工作量大；③ 回报是 CI 与应用共享同一套 RBAC/配额/GitOps 体系。

### 4.5 内置环境变量速查（最高频的一张小抄）

| 信息 | GHA | GitLab CI | Jenkins | CircleCI |
|---|---|---|---|---|
| 当前提交 SHA | `$GITHUB_SHA` | `$CI_COMMIT_SHA` | `$GIT_COMMIT` | `$CIRCLE_SHA1` |
| 当前分支/引用 | `$GITHUB_REF_NAME` | `$CI_COMMIT_REF_NAME` | `$BRANCH_NAME` | `$CIRCLE_BRANCH` |
| 仓库路径 | `$GITHUB_REPOSITORY` | `$CI_PROJECT_PATH` | `$JOB_NAME` | `$CIRCLE_PROJECT_REPONAME` |
| 本次构建编号 | `$GITHUB_RUN_NUMBER` | `$CI_PIPELINE_IID` | `$BUILD_NUMBER` | `$CIRCLE_BUILD_NUM` |
| 事件/来源 | `${{ github.event_name }}` | `$CI_PIPELINE_SOURCE` | 插件提供 | `$CIRCLE_PR_NUMBER` 等 |
| "我正在 CI 里"标志 | `$GITHUB_ACTIONS=true` | `$GITLAB_CI` | `$JENKINS_URL` | `$CIRCLECI=true` |

---

## 5. 陌生工具 30 分钟上手法（通用 Checklist）

换工作/接项目遇到没见过的 CI 系统时，不要通读文档。按下面顺序查，30 分钟内必能提交第一条可用流水线：

1. **定位配置文件（2 分钟）**
   搜：`.gitlab-ci.yml` / `Jenkinsfile` / `.circleci/` / `azure-pipelines.yml` / `bitbucket-pipelines.yml` / `tekton/` 目录。找到了就知道归属。
2. **跑官方 Hello World（8 分钟）**
   新建分支，写一条只有 `echo` 的最小流水线并触发（push 或手动按钮），确认"我能让机器为我执行命令"。
3. **对照五要素表（5 分钟）**
   在该工具文档里确认：触发器写法、job/step 语法、执行环境（镜像/标签）、条件、多 job 依赖。本文 §1.3 的表覆盖了 90% 的检索。
4. **摸清三件套（8 分钟）**
   - 密钥：在哪个 UI 配置、YAML 里怎么引用？
   - 缓存：key 怎么定义、缓存哪个目录？
   - 制品：怎么上传、下游 job 怎么下载、保留多久？
5. **找一个现成模板抄（5 分钟）**
   看同公司已有的流水线 / 官方模板库 / 可复用组件市场（Marketplace、Include 模板、orbs、Hub、Shared Library）。**先抄通，再求懂**。
6. **确认本地调试与限速（2 分钟）**
   有无 CLI 本地校验（`act`、`glab ci lint`、`circleci config validate`、`tkn`）？套餐并发上限多少？避免排队半天。

完成这 6 步后，你只是"词汇量"不够，语法随时查；而**编排思想（DAG、缓存键设计、制品边界、最小权限）是完全通用的**。

---

## 6. 工具无关的最佳实践

这些原则在五家工具里全部成立，也是高级与初级使用者的真正差距：

1. **配置即代码，随仓库走**。流水线文件和代码一起 review、一起回滚；不要在 UI 上点出来没人能复现的构建。
2. **快速失败**。先跑耗时短、失败率高的检查（lint → 单测 → 集成测试 → 构建 → 部署），把最贵的步骤放最后。
3. **环境不可假设**。所有工具依赖（语言版本、系统包、CLI）都在流水线里显式安装或固定镜像标签；同一台 runner 上一次构建装了什么与你无关。
4. **幂等且可重入**。任何一步重跑不应产生脏状态（部署脚本可重复执行、发布号冲突能处理）。
5. **最小权限**。GHA 显式写 `permissions:`；密钥按环境（staging/production）拆分；生产部署加人工审批门（environment protection / approval job / when: manual）。
6. **固定版本**。action/镜像用主版本 tag（`@v4`）或 digest，不要用浮动的 `@main`，避免某天上游变更让全公司 CI 红掉。
7. **日志面向排障**。关键步骤打印版本号与输入参数；失败日志能让人不看代码就定位问题。
8. **把缓存当优化、不当依赖**。缓存随时可能失效（逐出、换 runner），流水线在"零缓存"时必须仍然正确。
9. **控制成本**。用 path 过滤（只改文档就别跑全套）、`cancel-in-progress` 取消过时运行、矩阵只保留必要组合、重活交给自托管 runner。
10. **流水线模板化**。超过 3 个仓库出现相似 YAML，就抽 reusable workflow / include / orb / Shared Library / Hub Task，并设版本号。

---

## 7. 跨工具共通的坑

| 坑 | 现象 | 通用解法 |
|---|---|---|
| 时区 | cron/日期在 UTC 与本地时间间错位 | 统一用 UTC 编排，展示层再换算 |
| 行尾符 | Windows runner 上 shell 脚本因 CRLF 报错 | `.gitattributes` 锁 `*.sh text eol=lf` |
| Job 间文件不共享 | "上一步明明生成了，下游却说找不到" | 跨 job 一律走制品/共享卷；同一 job 内才是同一台机器 |
| Fork PR 无密钥 | 外部贡献者的 PR 构建在部署步骤失败 | 设计为"PR 只做 CI（无密钥），合并后才做 CD" |
| 密钥被变形泄露 | base64/二次编码后绕过日志打码 | 永远通过环境变量注入，禁止打印/转换 |
| 浮动版本漂移 | 今早还绿的 CI 下午突然红 | 固定 action/镜像大版本，定期统一升级 |
| Flaky test | 偶发失败侵蚀对 CI 的信任 | 单独标记重试上限并统计 flaky 率，根因修复前不算通过 |
| 并发互踩 | 两个部署同时写同一环境 | concurrency group / resource_group / 串行队列 |
| 磁盘被缓存撑满 | 自托管 runner 越跑越慢甚至 No space left | 定期清理缓存目录、限制缓存总量、优先容器化一次性环境 |
| 隐式依赖宿主机 | 本地/老 runner 能过，全新 runner 失败 | 所有依赖写进流水线或镜像（见最佳实践 3） |
| 免费额度悄悄用尽 | 账单或配额告警 | 加用量监控/预算告警；大矩阵前先估算分钟数 |

---

## 8. 练习路线与自测

### 8.1 阶梯练习（建议每题都真实提交到仓库）

1. **L1**：push 触发，打印分支名与 SHA；
2. **L2**：任意语言项目跑通"装依赖 + lint + test"；
3. **L3**：加 Node/Python 双版本 + 双 OS 矩阵，失败不互相取消；
4. **L4**：配置依赖缓存，对比第二次运行耗时；
5. **L5**：构建产物上传，第二个 job 下载并打印文件清单；
6. **L6**：PR 跑 CI；push 到 main 才"部署"（先用 `echo` 模拟），密钥通过 secret 注入；
7. **L7**：把 test 部分抽成 reusable workflow，在第二个仓库调用；
8. **L8（迁移练习）**：把 L6 的流水线**不查本文**分别翻译成 GitLab CI 与 CircleCI 版本，再对照 §4 订正；
9. **L9（平台化）**：给团队项目做一个模板仓库 + 版本发布流程（打 tag → 构建 → 发布 Release/镜像）。

### 8.2 自测清单（能答上即"出师"）

- [ ] 不看资料画出五要素 + 六辅助机制的关系图；
- [ ] 说出"Job 之间为什么不能直接共享文件"及通用解法；
- [ ] 解释缓存 key 为什么要含 lockfile 哈希、`restore-keys` 解决什么；
- [ ] 说出 fork PR 为什么拿不到密钥，以及 CI/CD 分离的设计；
- [ ] 拿到一个陌生工具，能在 30 分钟内按 §5 清单提交第一条流水线；
- [ ] 能把同一段 GHA YAML 口述翻译成 GitLab CI 与 CircleCI 的结构（不要求逐字）；
- [ ] 说清什么时候该上 Tekton/Jenkins 这类自建方案，什么时候坚决用 SaaS（可参考配套报告第 7、8 章的选型框架）。

### 8.3 学完之后

- 选型与定价（个人/团队/企业各该用谁、成本怎么算）：见 [cicd-tools-comparison-2026.md](./cicd-tools-comparison-2026.md)；
- 同一任务在五家工具中的完整配置对照：见该报告 §4.1.1；
- 进阶方向：CD/GitOps（Argo CD 与 Tekton 组合）、流水线平台工程（模板治理、配额计费、runner 弹性供给）、CI 中的安全供应链（SBOM、签名、最小权限）。

---

*本教程语法以 2026-10 的 GitHub Actions 现行版本为准；示例刻意保持工具无关的通用结构，便于向任意 CI/CD 系统迁移。*
