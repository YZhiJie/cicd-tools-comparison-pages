# 主流 CI/CD 工具对比分析报告

> **数据截止**：2026-10-14（所有定价、限额、版本数据均以该日期访问官方页面核实为准）
> **核心对象**：GitHub Actions、GitLab CI/CD；**参照对象**：Jenkins、CircleCI、Tekton（另简评 Azure DevOps Pipelines、Bitbucket Pipelines、Buildkite、Argo、Drone/Travis CI）
> **标注约定**：定量数据均附 `[cite:N]`（来源见附录）；`[存疑]` = 来源冲突或口径不清；`[未查到]` = 无可靠公开来源；推断性结论附置信度（高/中/低）。

---

## 0. 执行摘要

**市场格局**：CI/CD 已从"独立工具竞争"演变为"代码托管平台生态之争"。JetBrains 2025 开发者生态调查（n=24,534）显示组织侧采用率 GitHub Actions 33%、Jenkins 28%、GitLab CI 19% [cite:45]；同口径下 Jenkins 由 2022 年的 47% 下滑至 28%，GitHub Actions 由 27% 升至 33%，呈明确的"一升一降"趋势 [cite:45][cite:51]。

**一句话结论**：

| 你的情况 | 首选 | 理由 |
|---|---|---|
| 代码在 GitHub，个人/开源/中小团队 | **GitHub Actions** | 公有库免费、私有库 2,000 分钟/月起、Marketplace 生态最大、零运维 [cite:1][cite:13] |
| 需要一体化 DevSecOps（代码+CI+安全+合规） | **GitLab CI** | 单应用全覆盖，self-managed 满足数据主权，Premium $29/人/月 [cite:20][cite:27] |
| 存量大型企业、深度定制、内网隔离 | **Jenkins**（或 CloudBees） | 2,000+ 插件、完全自控，但要算清运维人力账 [cite:35][cite:52] |
| 追求构建性能与弹性、愿意按量付费 | **CircleCI** | credits 按量计费、resource class 粒度细、并发 80x 起 [cite:41] |
| 已有 Kubernetes 平台团队、要云原生标准化 | **Tekton** | K8s CRD 原生、2026-03 起 CNCF incubating，但门槛最高 [cite:38a][cite:39] |

**2025-2026 关键时效事件**（直接影响成本测算）：
1. GitHub Actions 托管 runner 自 2026-01-01 起**降价最高 39%**；原定 2026-03-01 对私有库 self-hosted runner 征收的 $0.002/分钟"云平台费"在社区反弹后约两日内宣布推迟，**截至 2026-10 从未生效** [cite:7][cite:8][cite:9]。
2. GitLab 自 18.7 起引入 **GitLab Credits** 按量计费（随 Duo Agent Platform，超额 $1/credit，Premium 每用户含 $12 credits），CI 分钟价格体系本身保持稳定 [cite:28]。
3. CircleCI Server（自托管版）**未停售**：4.10.x 持续维护（Helm 部署、支持 K8s 1.33-1.35、FIPS 140）[cite:43][cite:44]。
4. Tekton 于 2026-03 从 CDF 转入 **CNCF（incubating）**，治理归属发生变化 [cite:39]。

---

## 1. 研究范围与方法

**分析维度**（按需求设定）：① 核心架构与运行机制；② 适用场景与目标用户；③ 核心功能特性（配置语法 / 并发 / 自托管 runner / 缓存 / 生态）；④ 部署运维成本、学习曲线、许可费用；⑤ 优势、局限与典型案例；⑥ 分规模选型决策标准。

**信息源分级**：P0 = 官方文档/定价页/SEC 文件（本报告绝大多数定量数据来源）；P1 = 权威调查（JetBrains、CDF、CNCF）；P2 = 技术媒体；P3 = 第三方价格追踪站（仅作旁证并标注）。

**交叉验证**：核心定价与限额数据至少经"官方页面 + 独立来源"双重核实；GitHub Actions / GitLab CI / CircleCI 的价格表由研究代理与报告作者分别独立访问官方页面各核实一次（2026-10-14）。

**已知数据缺口**：Stack Overflow 开发者调查不设 CI/CD 工具题项（2025 版 n=49,009，仅有 Docker/K8s 等邻近数据）[cite:49]；GitHub Marketplace actions 精确数量官方口径停留在 "10,000+"，第三方称 2–3 万 [存疑] [cite:13]；CDF《State of CI/CD》最新公开版为 2024 版，未见 2025 版 [未查到] [cite:50]。

---

## 2. 市场格局与采用趋势

### 2.1 采用率（第三方调查）

| 来源（年份，样本） | GitHub Actions | Jenkins | GitLab CI | 备注 |
|---|---|---|---|---|
| JetBrains 开发者生态 2025（n=24,534，组织侧） | **33%** | **28%** | 19% | 18% 组织不使用任何 CI/CD [cite:45] |
| JetBrains CI/CD 专项 2025（n=805） | 个人项目 **62%** / 组织 41% | — | — | 32% 组织同时使用 2 个 CI 工具、9% 用 3 个以上；73% 未在 CI/CD 中使用 AI [cite:46] |
| JetBrains 生态 2022（组织侧，趋势基线） | 27% | **47%** | — | 三年间 Jenkins -19pp、GHA +6pp [cite:51] |

**趋势判断（置信度：高）**：GitHub Actions 借 GitHub 平台绑定优势成为新增项目的默认选项；Jenkins 存量仍大但份额持续流失；GitLab CI 与其 SCM 客户基本盘强绑定。

### 2.2 厂商规模口径（截至 2026，各自官方数据，不可直接互比）

| 厂商 | 公开口径 | 来源 |
|---|---|---|
| GitHub | 1.8 亿+ 开发者、6.3 亿+ 仓库（Octoverse 2025） | [cite:16] |
| GitLab | 5,000 万+ 注册用户；50%+ Fortune 100 为客户；ARR>$5K 客户 10,682 家（FY2026 Q4） | [cite:47] |
| CircleCI | 宣称 200 万+ 用户；Gartner 2025 DevOps 平台魔力象限 Niche Player | [cite:48] |
| Jenkins | 2019 年官方口径 25 万+ 活跃安装（历史数据，此后未更新） | [cite:52] |

### 2.3 格局分类与含义

市场呈 **"双生态主导 + 存量巨头 + 云原生新贵"** 结构：GitHub Actions 与 GitLab CI 依托 SCM 平台形成"代码在哪，CI 在哪"的引力；Jenkins 凭插件生态与可定制性守住大型/合规型组织存量；Tekton 以 K8s 标准件身份进入平台工程场景。CDF 2024 报告还发现一个对选型有直接影响的相关性：**同时使用托管 + 自托管 CI/CD 的混合团队 DORA 部署表现最佳**（第 5 期，SlashData 执笔；样本规模未公开 [存疑]）[cite:50]——"全押单一形态"并非最优解。

---

## 3. 核心架构与运行机制差异

### 3.1 四种架构范式

| 范式 | 代表 | 控制面 | 执行面 | 本质差异 |
|---|---|---|---|---|
| **SCM 内嵌事件驱动** | GitHub Actions、GitLab CI（SaaS） | 代码平台即控制面，事件（push/PR/tag）原生触发 | 托管临时 VM/容器 + 自托管 runner | CI 是代码平台的原生功能，零独立部署 |
| **独立控制器-代理** | Jenkins、GitLab self-managed、CircleCI Server | 自建的 controller/server | agent 主动连接 controller 领取任务 | 控制面完全自控，可内网/离线，运维自担 |
| **托管控制面 + 混合执行** | CircleCI Cloud、Buildkite | 厂商 SaaS 调度 | 云执行器 或 自托管 runner/agent | 元数据上云、计算可下沉 |
| **K8s 原生 CRD** | Tekton（Argo Workflows 同类） | K8s API server（无独立控制面进程） | 每个 step 是 Pod 中的容器 | 流水线即 K8s 资源对象，复用 K8s 调度/权限/生态 |

### 3.2 各工具机制剖面

**GitHub Actions**：workflow 以 YAML 存放于仓库 `.github/workflows/`，由 push、pull_request、schedule、workflow_dispatch 等事件触发；job 在 runner 上执行，自定义 action 分 JavaScript、Docker 容器、composite 三类 [cite:11]。托管 runner 为一次性干净环境；**self-hosted runner 主动发起出站 HTTPS（443）长连接拉取任务，无需开放入站端口**，官方明确其"非临时干净环境"，不建议用于公有库 fork PR 场景 [cite:5][cite:6]。

**GitLab CI**：流水线以仓库根目录 `.gitlab-ci.yml` 声明，支持 `stages` 顺序执行与 `needs` 构成的 DAG 并行；支持 merge request pipelines、merged results、merge train、parent-child（trigger）流水线 [cite:17]。GitLab Runner 为 Go 实现，当前主推 executor 为 Kubernetes、Docker、Docker Autoscaler、Instance（后两者基于 fleeting 自动扩缩）；SSH/Shell/Parallels/VirtualBox/docker+machine 已停止功能开发、仅维护安全更新 [cite:18]。SaaS hosted runner 每个 job 使用 GCP 上一次性隔离 VM（small 2C/8G 至 2xlarge 32C/128G，大机型仅付费档可用）[cite:19]。

**Jenkins**：controller-agent 模型，Java 技术栈；流水线以 Jenkinsfile（Declarative 或 Scripted，基于 Groovy）定义；可通过 Configuration as Code（JCasC）管理控制器配置本身；Kubernetes 插件可在 K8s 上动态供给一次性 agent。LTS 基线每 12 周一个、每 4 周补丁；**2026-01 发布的 LTS 2.541.1 起支持 Java 25，2.555.1 起要求 Java 21/25（Java 17 已弃用）** [cite:36][cite:37][cite:38b]。

**CircleCI**：云原生 SaaS，`config.yml`（2.1 语法）定义 pipeline，支持 dynamic configuration（按路径/参数生成后续配置）；executor 覆盖 Docker（x86/Arm）、Linux/Windows VM、macOS（M4 Pro）、GPU（仅 Scale 档）；自托管 runner 支持 machine 与 container（K8s）两种形态 [cite:41]。Server 版以 Helm chart 部署在客户自有 K8s 上（4.10.x，支持 K8s 1.33-1.35）[cite:43][cite:44]。

**Tekton**：以 Kubernetes CRD 实现——`Task`/`Pipeline` 为定义、`TaskRun`/`PipelineRun` 为执行实例，事件触发由 Triggers 组件（EventListener/TriggerTemplate）承担，每个 step 以容器运行于 Pod [cite:38a]。前身为 Knative Build，2019 年捐给 CDF（首批 graduated 项目），**2026-03 转入 CNCF incubating**；月度发布，截至 2026-10 最新 v1.17.0，最新 LTS v1.12.0（支持至 2027-05）[cite:39][cite:40]。Red Hat OpenShift Pipelines、IBM Cloud Continuous Delivery、Jenkins X（2026-04 更名 JayeX）、GKE 上的 Kf 均以 Tekton 为底座 [cite:39][cite:42]。

### 3.3 架构维度横向对比

| 维度 | GitHub Actions | GitLab CI | Jenkins | CircleCI | Tekton |
|---|---|---|---|---|---|
| 独立部署控制面 | ❌（绑定 GitHub/GHES） | ⚠️ SaaS 无需 / self-managed 需部署 | ✅ 必须自建 | ⚠️ Cloud 无需 / Server 自建 | ✅ 必须有 K8s 集群 |
| 执行环境默认隔离 | ✅ 每 job 一次性 VM | ✅ 每 job 一次性 VM（hosted） | ❌ 默认复用 agent（需自行容器化） | ✅ 容器/VM 按 job 供给 | ✅ 每 step 容器级隔离 |
| 配置即代码 | ✅ YAML 随仓库 | ✅ YAML 随仓库 | ✅ Jenkinsfile（Groovy） | ✅ YAML 随仓库 | ✅ YAML（K8s manifest） |
| 配置复用机制 | actions / reusable workflows | `include` / CI/CD Catalog | Shared Libraries | orbs | Tekton Hub / Bundle |
| 内网/离线可用 | ⚠️ 仅 GHES | ✅ self-managed | ✅ 完全支持 | ✅ Server | ✅ 完全支持 |
| 调度基础设施依赖 | 无（SaaS） | 无（SaaS）/ 自备 runner | JVM + 控制器存储 | 无（Cloud） | Kubernetes（硬前提） |

---

## 4. 核心功能特性对比

### 4.1 流水线配置语法

| 维度 | GitHub Actions | GitLab CI | Jenkins | CircleCI | Tekton |
|---|---|---|---|---|---|
| 语言/格式 | YAML | YAML | Groovy DSL（Declarative/Scripted） | YAML | YAML（K8s CRD） |
| 并行模型 | job 默认并行 + matrix | stages 顺序 + `needs` DAG | `parallel` 指令 | workflows 内 DAG | `runAfter` DAG |
| 条件与动态化 | `if` 表达式、reusable workflow | `rules`/`workflow:rules`、parent-child | 完整编程能力（Groovy） | dynamic configuration（setup pipeline） | 参数化 + when 表达式 |
| 学习门槛 | 低（YAML + 表达式） | 低-中（关键字体系丰富） | **高**（Groovy + 插件 API） | 低-中 | **高**（需先懂 K8s） |
| 本地调试 | 弱（靠 act 等第三方） | 弱（`glab ci run` 部分覆盖） | 中（Replay/Blue Ocean 重跑） | 中（CLI 本地执行） | 中（`tkn` CLI） |

最小示例观感对比（同为"安装依赖并测试"）：

```yaml
# GitHub Actions：.github/workflows/ci.yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

```yaml
# GitLab CI：.gitlab-ci.yml
test:
  stage: test
  image: node:22
  script: ["npm ci", "npm test"]
```

```groovy
// Jenkins：Jenkinsfile（Declarative）
pipeline {
  agent { docker 'node:22' }
  stages { stage('test') { steps { sh 'npm ci && npm test' } } }
}
```

```yaml
# CircleCI：.circleci/config.yml
jobs:
  test:
    docker: [{ image: cimg/node:22 }]
    steps: [checkout, { run: "npm ci && npm test" }]
workflows: { ci: { jobs: [test] } }
```

```yaml
# Tekton：Task（节选）——必须显式定义 workspace/step，冗长但完全声明式
spec:
  steps:
    - name: test
      image: node:22
      script: npm ci && npm test
```

**要点**：GitHub Actions 与 GitLab CI 语法最"傻瓜"，差异在调度模型（GHA 默认全并行、GitLab 默认按 stage 串行）；Jenkins 的 Groovy 带来几乎无限的表达力，也意味着流水线里容易长出"只有原作者看得懂的代码"；Tekton 的表达完全是 K8s 风格，声明同样的事情 YAML 行数最多，但换来 K8s 原生的一致性。

#### 4.1.1 同一任务五套写法对照（矩阵测试 + 缓存 + 产物）

上面的最小示例只展示了"跑一条命令"。为了把各家差异拉满，下面用**同一条真实流水线**各写一遍：push/PR 触发 → Node 20/22 双版本矩阵 → npm 缓存 → 上传覆盖率产物。五种配置哲学（"一切皆 step" / "stage 先行 + rules 过滤" / "Groovy 即编程" / "job 定义与编排分离，参数化是灵魂" / "流水线即 K8s 资源对象"）在这一组对照中一目了然。

**① GitHub Actions**（`.github/workflows/ci.yml`）：

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: npm
      - run: npm ci
      - run: npm test
      - uses: actions/upload-artifact@v4
        with:
          name: coverage-${{ matrix.node }}
          path: coverage/
```

**② GitLab CI**（`.gitlab-ci.yml`）：

```yaml
stages: [test]

test:
  stage: test
  image: node:22
  parallel:
    matrix:
      - NODE_VERSION: ["20", "22"]
  script:
    - npm ci --cache .npm
    - npm test
  cache:
    key: npm-$NODE_VERSION
    paths: [.npm]
  artifacts:
    paths: [coverage/]
    expire_in: 7 days
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
```

注意：**GitLab 自动 clone 仓库**，没有 checkout 步骤；触发条件不写在文件头部，而是落在每个 job 的 `rules` 里。

**③ Jenkins**（`Jenkinsfile`，Declarative）：

```groovy
pipeline {
  agent none
  stages {
    stage('test') {
      matrix {
        axes { axis { name 'NODE_VERSION'; values '20', '22' } }
        agent { docker { image "node:${NODE_VERSION}" } }
        stages {
          stage('install & test') {
            steps {
              checkout scm                       // 必须显式拉代码
              sh 'npm ci --cache .npm --prefer-offline'
              sh 'npm test'
            }
          }
        }
      }
    }
  }
  post {
    always { archiveArtifacts artifacts: 'coverage/**', fingerprint: true }
  }
}
```

注意：Jenkins **没有内建缓存关键字**（要么持久化 workspace，要么靠插件/共享存储）；触发由 Webhook 插件承担，不写在配置文件里。

**④ CircleCI**（`.circleci/config.yml`）：

```yaml
version: 2.1
jobs:
  test:
    parameters:
      node-version: { type: string }
    docker:
      - image: cimg/node:<< parameters.node-version >>
    steps:
      - checkout
      - restore_cache: { keys: ['npm-{{ checksum "package-lock.json" }}'] }
      - run: npm ci
      - save_cache:
          key: npm-{{ checksum "package-lock.json" }}
          paths: [.npm]
      - run: npm test
      - store_artifacts: { path: coverage/ }
workflows:
  ci:
    jobs:
      - test:
          matrix: { parameters: { node-version: ["20", "22"] } }
```

注意：缓存是**显式的 restore/save 两步**，key 支持按 lockfile 内容哈希（`{{ checksum }}`），依赖锁定最精确；矩阵通过 workflow 层向参数化 job 传值实现。

**⑤ Tekton**（Task 定义，触发与矩阵在配置之外）：

```yaml
apiVersion: tekton.dev/v1
kind: Task
metadata: { name: node-test }
spec:
  params: [{ name: node-version, default: "22" }]
  workspaces: [{ name: source }]
  steps:
    - name: test                       # git-clone Task 已在前面把代码写入 workspace
      image: node:$(params.node-version)
      script: |
        npm ci --cache .npm
        npm test
```

注意：Tekton **没有"触发器"和"矩阵"的内建概念**——push 触发由 EventListener/Triggers 组件负责，矩阵靠外部循环生成多个 PipelineRun；缓存与产物均需自行用 workspace（PVC）或对象存储拼装。

**配置要素逐项对照**：

| 配置要素 | GitHub Actions | GitLab CI | Jenkins | CircleCI | Tekton |
|---|---|---|---|---|---|
| 配置文件位置 | `.github/workflows/*.yml` | `.gitlab-ci.yml` | `Jenkinsfile` | `.circleci/config.yml` | K8s manifest（可拆多文件） |
| 拉取代码 | 显式（checkout action） | **自动 clone** | 显式（`checkout scm`） | 显式（checkout step） | 显式（git-clone Task） |
| 触发器声明 | 文件头 `on:` | job 级 `rules` | Webhook 插件（文件外） | workflow `filters` | Triggers 组件（独立 CRD） |
| 条件语法 | `if` + `${{ }}` 表达式 | `rules: if/changes/exists` | `when` + Groovy | `when` / `filters` | `when` 表达式 |
| 矩阵构建 | `strategy.matrix`（最简） | `parallel: matrix` | `matrix` axes（冗长） | workflow 级 `matrix` + 参数 | ❌ 无原生，需外部生成 |
| 缓存 | `actions/cache` 或 setup-* 内建 | `cache:` 关键字 | ❌ 无内建 | restore/save 两步 + `checksum` | ❌ 无内建 |
| 构建产物 | upload/download-artifact | `artifacts:` 关键字 | `archiveArtifacts` | `store_artifacts` | workspace / Results |
| 密钥引用 | `${{ secrets.X }}` | `$CI_VAR`（UI/文件配置） | `withCredentials()` | Contexts + 环境变量 | K8s Secret 挂载 |
| 变量插值 | `${{ }}` 模板 | `$VAR`（shell 级） | `"${VAR}"`（Groovy 级） | `<< param >>` / `$VAR` | `$(params.x)` 替换 |
| 配置复用 | reusable workflows + composite actions | `include:` + CI/CD Catalog | Shared Libraries | orbs | Tekton Hub / Bundle |
| 默认并行语义 | **job 间默认并行** | **stage 间默认串行**（`needs` 改 DAG） | 串行，`parallel` 显式声明 | workflow DAG 显式声明 | DAG `runAfter` 显式声明 |

**实践含义**：

- 追求"声明一次、到处复制"的成本最低 → GitHub Actions 的 step 模型 + 庞大 action 生态，让同一套逻辑改个 trigger 就能跑；
- 需要精确控制"什么情况跑哪个 job"（分支、路径、变量组合）→ GitLab 的 `rules` 体系表达力最强，但心智负担也最高；
- 流水线里存在复杂业务逻辑（动态生成 stage、跨 job 传递对象、条件回退）→ 只有 Jenkins 的 Groovy 做得到，代价是可维护性与"作者锁死"风险；
- 需要按 lockfile 哈希精确失效的依赖缓存 → CircleCI 的 `{{ checksum }}` 设计最干净，其余各家都能实现但写法更啰嗦；
- 团队已全面 Kubernetes 化 → Tekton 的冗长换来的是完全的 K8s 一致性（RBAC / 配额 / NetworkPolicy / GitOps 可全套复用）。

### 4.2 并发执行能力（as of 2026-10）

| 工具 | 托管并发上限 | 自托管并发 | 弹性扩缩 |
|---|---|---|---|
| GitHub Actions | 标准 runner 总并发：Free 20 / Pro 40 / Team 60 / Enterprise 500；其中 macOS 分别 5/5/5/50；larger runner Team/Enterprise 1,000（GPU 100）[cite:3] | 不限（自管容量） | larger runner 支持自动扩缩；自托管有 Actions Runner Controller（ARC，K8s） |
| GitLab CI（SaaS） | 不按并发售卖，瓶颈为 compute 配额与 job 限额：活动流水线 job 上限 Free 500 / Premium 20,000 / Ultimate 60,000；单流水线 job 上限 500/1,500/2,000 [cite:29] | `config.toml` 的 `concurrent` 参数控制全局并发 [cite:18] | Docker Autoscaler / Instance executor（fleeting，支持 AWS/GCP/Azure）、K8s executor [cite:18] |
| Jenkins | 无托管版 | 完全自管（agent 数 × executor 数） | K8s 插件动态 agent、EC2/Azure 等云插件 |
| CircleCI | 按套餐：Free 30x / Performance 80x / Scale 不限；macOS 例外：1x/15x/定制 [cite:41] | 每 runner 并发任务数：Free 5 / Performance 20 / Scale 不限 [cite:41] | 按量计费天然弹性；Server 版用 Nomad Autoscaler [cite:43] |
| Tekton | 无托管版 | 受 K8s 集群容量与配额限制 | 原生依赖 K8s 调度 + Cluster Autoscaler/KEDA |

### 4.3 自托管 Runner 支持对比

| 维度 | GitHub Actions | GitLab CI | Jenkins | CircleCI | Tekton |
|---|---|---|---|---|---|
| 自托管形态 | runner 应用（出站长连接）[cite:5] | GitLab Runner（Go，多 executor）[cite:18] | agent（JNLP/SSH） | machine / container runner | 天然"全自托管" |
| 通信方向 | 出站 443，无入站 [cite:5] | 出站轮询 | 双向（可 controller 推） | 出站 | 集群内 |
| 隔离性 | ❌ 非临时环境，需自行加固 [cite:6] | ⚠️ 取决于 executor（Docker/K8s 可一次性） | ❌ 默认复用 | ⚠️ machine 可配一次性 | ✅ Pod 级 |
| 计费（2026-10 现行） | **免费**（$0.002/min 收费已推迟未生效）[cite:8][cite:9] | 免费（不计 compute 配额）[cite:17] | 免费（开源） | 不按分钟计费，仅缓存/workspace 回传出网超量按 420 credits/GB 计 [cite:41] | 免费（开源） |

### 4.4 缓存与产物策略

| 维度 | GitHub Actions | GitLab CI | Jenkins | CircleCI | Tekton |
|---|---|---|---|---|---|
| 依赖缓存 | `actions/cache`；每仓库 **10GB** 上限，超限按最近访问时间逐出，7 天未访问清除；超额 $0.07/GB/月；低信任触发器只读默认分支缓存防投毒 [cite:10][cite:1] | `cache:` 默认存 runner 本地；autoscale 需配分布式缓存（S3/GCS/Azure）[cite:30] | 无内建标准（靠插件/共享存储/自行设计） | `save_cache`/`restore_cache`；存储超量 420 credits/GB（≈$0.252/GB）[cite:41] | 无内建，靠 workspace（PVC）自行组织 |
| Docker 层缓存 | 需自配（buildx + cache backend） | 需自配 | 需自配 | **DLC 内建**，200 credits/job（≈$0.12/job）[cite:41] | 需自配 |
| 构建产物 | artifacts，免费档 500MB 存储起 [cite:1] | artifacts 单 job 上限 1GB（压缩）、默认保留 30 天 [cite:29] | 自行管理（控制器磁盘/外部存储） | artifacts + workspace（存储 2GB 起）[cite:41] | 自行管理（对象存储/PVC） |

### 4.5 生态集成丰富度

| 维度 | GitHub Actions | GitLab CI | Jenkins | CircleCI | Tekton |
|---|---|---|---|---|---|
| 可复用组件库 | Marketplace 官方口径 **10,000+** actions（第三方称 2–3 万 [存疑]）[cite:13] | CI/CD Catalog + `include` 模板，规模较小 | **2,000+** 插件（索引页口径；首页口径 2,300+）[cite:35] | orbs 公共注册表（未查到官方计数 [未查到]） | Tekton Hub |
| 平台内集成 | PR 检查、CodeQL/Advanced Security、Packages、Pages、Codespaces 原生一体 [cite:13] | **一体化 DevSecOps 单应用**：SCM+CI+SAST/DAST+合规+价值流 [cite:32] | 无平台，全靠插件拼装 | 专注 CI/CD 单点，测试洞察（Insights/ flaky test）见长 [cite:41] | 与 K8s 生态天然一体（RBAC/Quota/NetworkPolicy） |
| 第三方接入 | 极丰富 | 中（内建为主） | 极丰富但质量参差 | 中 | 偏底层 |

**判断（置信度：中）**：论"现成组件数量"GitHub Marketplace 居首；论"开箱即用的深度"GitLab 内建功能最全（安全扫描、合规无需第三方）；Jenkins 插件数量与质量方差最大——既是最大资产也是最大负担。

---

## 5. 成本模型对比（许可 / 定价 / 运维 / 学习曲线）

### 5.1 许可与定价总表（as of 2026-10，均为官方公开价）

| 工具 | 许可/模式 | 免费档 | 入门付费 | 旗舰/企业档 | 计费单元 |
|---|---|---|---|---|---|
| GitHub Actions | 平台功能（随 GitHub 套餐） | 公有库免费；私有库 2,000 分钟/月 + 500MB 存储（Free）[cite:1] | Team $4/人/月（3,000 分钟）[cite:14][cite:15] | Enterprise $21/人/月（50,000 分钟）[cite:15] | 按构建分钟（OS 倍率） |
| GitLab CI | 开源核心 + 订阅（SaaS/self-managed 同价） | Free：400 compute 分钟/月 [cite:20] | Premium **$29/人/月**（10,000 分钟）[cite:20][cite:27] | Ultimate **$99/人/月**（50,000 分钟）[cite:20][cite:23] | compute minutes × 成本系数 |
| Jenkins | **MIT 开源** | 全功能免费 | CloudBees CI 定制报价（第三方估算中型团队 $10K–50K+/年 [P3，未独立核实]）[cite:53] | CloudBees 平台（Forrester TEI 为厂商委托研究，宣称 ROI 426%，仅作参考）[cite:52b] | 无许可费，成本=基础设施+人力 |
| CircleCI | SaaS 订阅 + credits | 30,000 credits/月（small Docker 约 6,000 分钟）、5 用户、30x 并发 [cite:41] | Performance **$15/月起**（+ $15/25,000 credits；80x 并发）[cite:41] | Scale 定制（年付，GPU/无限并发）[cite:41] | credits（资源类别 × 分钟 + 用户 + 增值项） |
| Tekton | Apache 2.0 开源 | 全功能免费 | — | 商业底座：OpenShift Pipelines 等随平台订阅 | 无许可费，成本=K8s 集群+平台人力 |
| Azure DevOps Pipelines（参照） | SaaS | 1 个托管并行作业（1,800 分钟/月）+ 1 个自托管作业（不限分钟）[cite:54] | 额外托管作业 $40/个/月 | 自托管作业 $15/个/月 [cite:54] | 按并行作业数 |
| Bitbucket Pipelines（参照） | SaaS | Free 50 分钟/月 | Standard 2,500 分钟/月 | Premium 3,500 分钟/月 [cite:55] | 按构建分钟 |
| Buildkite（参照） | 混合 SaaS | 免费档 5 用户、10 并发作业 | Pro 约 $20–30/活跃用户/月（定价页存在两个版本，按区间表述）[cite:56] | Enterprise 定制 | 按活跃用户 |

### 5.2 超额用量单价（托管 Linux 构建，as of 2026-10）

| 工具 | 超额单价 | 说明 |
|---|---|---|
| GitHub Actions | Linux 2C **$0.006/分钟**；arm64 $0.005；Windows $0.010；macOS $0.062；ubuntu-slim $0.002；larger runner 4C $0.012 ～ 96C $0.252；GPU $0.052/$0.102 [cite:2] | 公有库标准 runner 免费；larger runner 不可用免费额度 [cite:1] |
| GitLab CI | **$10 / 1,000 compute 分钟**（≈$0.010/分钟，一次性购买、12 个月有效）[cite:21] | 成本系数：Linux small=1、medium=2、large=3、xlarge=6、2xlarge=12、GPU medium=7、macOS M1=6/M2 Pro=12、Windows=1 [cite:17] |
| CircleCI | Linux medium 10 credits/分钟，credits $15/25,000 → 折合约 **$0.006/分钟**；大额预购 100 万 credits $600（折合 $0.0036/分钟）[cite:41] | macOS M4 Pro、GPU 等按各自 credits 率另计 |
| Jenkins / Tekton | —（自建基础设施成本） | 云主机+存储+网络，无平台加价 |

### 5.3 2025–2026 定价动向（时效性最强的一节）

1. **GitHub Actions**：2025-12-16 宣布 2026-01-01 起托管 runner 降价最高 39%（已生效，上表即现行价）；同时宣布原定 2026-03-01 对私有库 self-hosted runner 收 $0.002/分钟"云平台费"，遭社区强烈反弹后约两日内（2025-12-18 前后）宣布推迟并重新评估。**截至 2026-10-14，官方计费文档仍写明 self-hosted 免费，该费用从未生效**；公有库与 GHES 自始至终不受影响 [cite:7][cite:8][cite:9]。→ 选型含义：self-hosted runner 的"永久免费"承诺已被打破过一次，重度依赖者应把"未来可能收费"写入风险评估（置信度：中——官方措辞为"推迟"而非"取消"）。
2. **GitLab**：CI 分钟体系（400/10K/50K、$10/1000）自 2020-2022 年收紧后保持稳定 [cite:25][cite:26]；Premium 2023-04-03 由 $19 涨至 $29 [cite:27]；Ultimate $99 自 2017 年来未见公开涨价 [未查到调价记录]。2025-2026 年新变量是 **GitLab Credits**（18.7 引入，Duo Agent Platform 的按量货币，超额 $1/credit、Premium 每用户含 $12）——AI 功能开始走消耗制 [cite:28]。
3. **CircleCI**：credits 模型稳定；免费档 credits 当月过期不结转，付费 credits 12 个月有效 [cite:41]。
4. **GitHub/GitLab 套餐价对比**：同为入门付费档，GitHub Team $4/人/月 vs GitLab Premium $29/人/月——但二者不可直接比较：GitHub 的 CI 分钟主要按量另算，GitLab Premium 的 $29 同时买了 SCM 高级功能+CI 额度+一体化能力（置信度：高，定价结构差异所致）[cite:14][cite:20]。

### 5.4 部署与运维成本（TCO）

| 工具 | 初始部署 | 持续运维负担 | 隐性成本点 |
|---|---|---|---|
| GitHub Actions（SaaS） | 零 | 极低 | 矩阵构建 × macOS 10 倍单价放大数据；自托管 runner 需自行加固与更新 [cite:3][cite:6] |
| GitLab CI（SaaS） | 零 | 低 | 大机型 compute 系数放大消耗；self-managed 版升级、备份、Runner 维护需专职 |
| Jenkins | 中-高（控制器+agent+插件选型） | **高**：插件升级兼容性、安全补丁、控制器 HA 均需人力；官方与社区持续投入现代化（2025 年 UI 重设计、Java 25 迁移）[cite:36][cite:38b] | "插件地狱"与升级冻结风险；插件质量无人兜底 |
| CircleCI（Cloud） | 零 | 低 | credits 消耗需监控（DLC 每 job 200 credits、IP ranges 450 credits/GB 等增值项）[cite:41] |
| Tekton | **高**（需先有健康 K8s 集群） | 高：控制器升级、CRD 版本演进、触发器/权限治理 | 把 CI 问题变成了 K8s 平台问题；对无 K8s 团队的"隐性前提成本"最高 |

### 5.5 学习曲线（定性，基于语法与运维模型）

**GitHub Actions ≈ CircleCI < GitLab CI < Jenkins < Tekton**

- GitHub Actions / CircleCI：纯 YAML + 少量表达式，文档与示例密度高，数小时上手。
- GitLab CI：YAML 但关键字体系庞大（rules、needs、interruptible、resource_group…），进阶复杂度上来得快。
- Jenkins：除 YAML 式 Declarative 外，深度使用绕不开 Groovy 与插件 API 心智负担，管理员还需掌握 JCasC、权限矩阵。
- Tekton：前提是理解 Pod/CRD/RBAC/触发器，对非 K8s 团队是"先学一座山再爬山"。

### 5.6 情景成本测算（估算示例，假设显式列出）

> 场景：20 人团队，每月 10,000 分钟 Linux 构建（medium 规格），使用 SaaS 托管。**以下为按公开价目做的算术估算，非厂商报价**。

| 方案 | 计算 | 月成本估算 |
|---|---|---|
| GitHub Actions（Team） | 席位 20×$4 = $80；分钟 (10,000−3,000)×$0.006 = $42 | **≈$122** [cite:2][cite:14] |
| GitLab（Premium） | 20×$29（含 10,000 compute 分钟，刚好覆盖） | **$580** [cite:20] |
| CircleCI（Performance） | 基础 $15；credits：计算 10,000 分钟×10 = 100,000，额外用户 15×25,000 = 375,000，合计 475,000−30,000(免费) = 445,000 credits ×($15/25,000) ≈ $267 | **≈$282** [cite:41] |
| Jenkins（自建，以云主机粗估） | 2 台 8C16G 按需实例 + 存储 ≈ $300–500，**不含** 0.2–0.5 个专职运维人力 | **$300–500+ 人力**（基础设施价因云而异 [估算，未逐一核价]） |

**解读（置信度：中）**：纯 CI 用量视角 GitHub Actions 单价最低；但 GitLab Premium 的 $580 同时替代了"SCM 付费档 + 安全扫描 + 合规"等若干独立工具的开销；Jenkins 账面基础设施不贵，真实 TCO 取决于运维人力折算。按分钟单价敏感的超大规模构建，自托管 runner（三大家均免费）是共同的成本逃逸舱。

---

## 6. 优势、局限性与典型案例

### 6.1 GitHub Actions

**优势**：① 与 GitHub 仓库/PR/安全检查原生一体，上手成本全场最低（个人项目采用率 62% 即为佐证 [cite:46]）；② 公有库免费 + 降价后的托管单价（Linux $0.006/分钟）极具攻击性 [cite:2]；③ Marketplace 组件数量第一 [cite:13]；④ larger runner 规格上限高（x64 96 核 / arm64 64 核 / GPU / macOS M2 Pro），GitHub 自用其构建 GitHub.com：1 小时内 15,000 个 CI job、15 万核（2023-09）[cite:4][cite:12]。
**局限**：① 矩阵上限 256 jobs/工作流、单 job 上限 6 小时 [cite:3]；② macOS 单价约为 Linux 10 倍，矩阵场景成本放大明显 [cite:2]；③ self-hosted runner 非隔离环境，官方不建议承载公有库 fork PR [cite:6]；④ 原生可观测性较弱（用量分析需导出）[未查到内置监控 SLA]。
**典型案例**：GitHub 自身（GitHub.com 全量 CI）[cite:12]；python/cpython、microsoft/vscode 等顶级开源项目（仓库内 `.github/workflows` 可公开核实）。

### 6.2 GitLab CI

**优势**：① 一体化 DevSecOps——SCM、CI、安全扫描、合规、价值流分析单应用交付，KnowBe4 用 Ultimate 实现日产部署 5 倍提升、CACI 工具链 7 合 1 且安全扫描快 13 倍 [cite:32][cite:33]；② self-managed 完整功能可完全私有部署，满足数据主权（哔哩哔哩经极狐 GitLab 落地超大规模私有化 CI/CD）[cite:34]；③ 流水线能力深（merge train、parent-child、DAG）[cite:17]；④ 自托管 runner 不计配额，成本可控 [cite:17]。
**局限**：① 免费额度历史收紧（2020 宣布 400 分钟、2022 覆盖公开项目）曾引发社区反弹，开源项目须加入 Open Source Program 方可豁免 [cite:25][cite:26][cite:24]；② 入门档 $29/人/月为五者最贵（vs GitHub Team $4）[cite:20][cite:14]；③ Windows/macOS hosted runner 仍为 Beta [cite:17]；④ 组件市场生态小于 GitHub。
**典型案例**：KnowBe4 [cite:32]、CACI [cite:33]、哔哩哔哩（极狐）[cite:34]。

### 6.3 Jenkins

**优势**：① 完全开源（MIT）+ 2,000+ 插件，可定制性天花板最高，任意老旧/冷门系统几乎都有现成插件 [cite:35]；② 可完全离线/内网运行，合规场景的最后选项；③ 社区治理成熟（CDF 创始项目），LTS 节奏稳定（12 周基线），持续现代化（2026-01 LTS 2.541.1 支持 Java 25、2025 年 UI 重设计）[cite:37][cite:38b][cite:36]。
**局限**：① 运维负担五者最重：插件兼容、安全补丁、控制器 HA 均需专职人力；② 默认 agent 复用、无隔离，构建卫生需自行容器化解决；③ Groovy 学习曲线与"流水线代码化失控"风险；④ 官方安装量口径停留在 2019 年的 25 万+，近年未见更新 [cite:52]，叠加 JetBrains 采用率 47%→28% 的下滑，增量市场地位持续走弱 [cite:51][cite:45]。
**典型案例**：eBay 早期以 Jenkins+Docker 交付全公司 CI 方案（jenkins.io 官方案例，年代较早）[cite:57]；大量金融/电信等传统大型企业的存量平台（JetBrains 2025 组织侧 28% 采用率可间接佐证 [cite:45]）。

### 6.4 CircleCI

**优势**：① 按量计费的资源粒度最细（x86/Arm Docker、VM、macOS M4 Pro、GPU），Linux 折合 $0.006/分钟、批量预购更低至约 $0.0036/分钟 [cite:41]；② 并发模型简单直接（Free 30x / Performance 80x / Scale 无限），self-hosted runner 不按分钟收费 [cite:41]；③ 开源项目额度慷慨：最高 400,000 credits/月（约 80,000 构建分钟，Linux/Arm/Docker）[cite:41]；④ 测试智能化（flaky test 检测、测试拆分）是差异化亮点 [cite:41]。
**局限**：① 2023-01-04 安全事件（客户 secrets 需轮换）对品牌信任的长期影响 [cite:58]；② credits 模型对成本可预测性不友好（DLC 200 credits/job、IP ranges 450 credits/GB 等增值项需精算）[cite:41]；③ 平台外生态（SCM 绑定、安全套件）弱于两大平台；④ Gartner 2025 定位 Niche Player，企业级"全家桶"叙事缺位 [cite:48]。
**典型案例**：Kajabi（从 Harness 迁出一年后回迁 CircleCI）[cite:59]；Axios（iOS 构建提速 25%）[cite:60]；ANA Systems（全日空客运系统整合）[cite:61]。

### 6.5 Tekton

**优势**：① K8s 原生标准件：复用集群调度/RBAC/配额，流水线可纳入 GitOps 统一管理；② 治理背书强：CDF graduated 后 2026-03 转入 CNCF incubating，迁移时 600+ 贡献者、11k+ star [cite:39]；③ 被 OpenShift Pipelines、IBM Cloud CD 等企业产品选作底座，B 端渗透走的是"被集成"路线 [cite:39][cite:42]；④ 版本节奏健康（月度发布，LTS 支持 1 年）[cite:40]。
**局限**：① 硬前提是已有 K8s 集群与相应技能，对简单场景明显过重（第三方文档直言初始配置复杂）[cite:62]；② 无 SaaS 托管版，Dashboard 为只读 UI，使用体验偏"半成品"，生产落地通常需要再套一层（OpenShift Pipelines/自建平台）；③ YAML 冗长，复用依赖 Hub/Bundle 纪律。
**典型案例**：Red Hat OpenShift Pipelines（商用产品化底座）[cite:42]；Jenkins X 自 2019 年起以 Tekton 为执行引擎（2026-04 更名 JayeX）[cite:39]。

### 6.6 其他参照工具速览

| 工具 | 一句话定位 | 关键事实（as of 2026-10） |
|---|---|---|
| Azure DevOps Pipelines | 微软生态企业的一体化选择 | 免费 1 托管并行作业（1,800 分钟）+1 自托管（不限分钟）；额外托管 $40/个、自托管 $15/个 [cite:54] |
| Bitbucket Pipelines | Atlassian 生态（Jira 联动）团队的默认项 | 按分钟分档：Free 50 / Standard 2,500 / Premium 3,500 分钟每月 [cite:55] |
| Buildkite | "SaaS 控制面 + 自托管 agent"混合派代表 | 按活跃用户订阅（约 $20–30/活跃用户/月，定价页存在 A/B 两版），免费档 5 用户 10 并发 [cite:56] |
| Argo（Workflows/CD） | GitOps 持续交付事实标准（不替代 CI 构建） | CNCF graduated（2022-12）；CNCF 2025-07 最终用户调查：多数采用的 GitOps 方案 [cite:63][cite:64] |
| Drone CI | 已被 Harness 收编，社区分叉延续 | 2020 年被 Harness 收购、许可证改 Polyform Small Business，社区分叉出 Woodpecker CI [cite:65] |
| Travis CI | 上一代开源 CI 王者，已边缘化 | 2019 年被 Idera 收购后声量持续下滑 [cite:66] |

---

## 7. 适用场景与目标用户

| 用户群体 | 首选 | 次选 | 不适用 | 关键考量 |
|---|---|---|---|---|
| **个人开发者 / 开源项目** | GitHub Actions（公有库免费、62% 个人采用率）[cite:46] | GitLab Free（400 分钟）、CircleCI 开源档（80K 分钟）[cite:41] | Jenkins/Tekton（自建不划算） | 零成本 + 零运维 + 社区示例密度 |
| **中小团队（5–50 人）** | 跟随代码托管：GitHub→Actions；GitLab→GitLab CI | CircleCI Performance（构建性能敏感时） | Tekton（除非已是 K8s 团队） | 管理面越少越好；成本随用量线性 |
| **大型企业（500+ 人）** | GitLab self-managed（数据主权+一体化）或 GHA Enterprise（GitHub 存量） | Jenkins（存量延续/特殊集成）、CircleCI Scale | — | 合规、SSO/审计、并发规模、厂商支持 SLA |
| **强监管/内网隔离（金融、政企、军工）** | Jenkins（完全离线） | GitLab self-managed、CircleCI Server（FIPS 140 支持）[cite:44] | 纯 SaaS 方案 | 离线可用、审计、国产化适配 |
| **K8s 平台工程团队** | Tekton（CI 标准件）+ Argo CD（GitOps 交付） | GitLab CI（K8s executor） | Jenkins 裸金属形态 | 声明式、可 GitOps 化、多租户隔离 |
| **iOS/macOS 构建密集团队** | GitHub Actions（macOS 并发 5–50、M 系机型） | CircleCI（M4 Pro、macOS 并发 15x）[cite:41] | Tekton（无 macOS 能力） | macOS 单价与并发配额是核心约束 |

**补充判断（置信度：中）**："代码托管在哪，CI 就用谁"已成为中小团队的最强决策因子——这也是 JetBrains 数据中 GHA/GitLab CI 采用率与其 SCM 份额高度同构的原因；只有性能、特殊硬件（GPU/macOS/Arm）或合规需求才会驱动团队脱离该引力。

---

## 8. 分规模选型决策框架

### 8.1 五步决策法

1. **代码托管在哪？** GitHub → 默认 GitHub Actions；GitLab → 默认 GitLab CI；Bitbucket → Bitbucket Pipelines 或 CircleCI；自托管 Git/多仓 → Jenkins 或 GitLab self-managed。平台引力能省掉的是集成与账号体系成本。
2. **有无合规/数据主权硬约束？** 有（内网、离线、等保/行业监管）→ 只剩 Jenkins、GitLab self-managed、CircleCI Server、Tekton 四个可选项；无 → 优先 SaaS。
3. **构建矩阵有无特殊硬件/OS？** GPU：CircleCI Scale / GHA larger runner / 自托管；macOS 密集：对比 GHA 与 CircleCI 的 macOS 并发配额与单价（10 倍于 Linux 的单价是共同痛点）[cite:2][cite:41]；Arm：三家托管均已覆盖。
4. **用量规模与成本模式偏好？** 低-中用量选按分钟计费（GHA/GitLab 免费额度内可能零成本）；高用量优先自托管 runner（三大家 self-hosted 均不计分钟费——注意 GHA 收费未遂事件带来的远期不确定性 [cite:9]）；用量波动大选 credits/按量（CircleCI）。
5. **团队技能与运维意愿？** 无专职平台团队 → 远离 Jenkins/Tekton；已有 K8s 平台团队且要标准化多租户 CI → Tekton；存量 Jenkins 且运转良好 → 不因追新而迁移（迁移成本高于工具差价，置信度：中）。

### 8.2 按团队规模推荐矩阵

| 规模 | 推荐主方案 | 预算参考（月度，估算） | 决策要点 |
|---|---|---|---|
| 1–5 人 | GitHub Actions Free / GitLab Free | $0 | 免费额度（2,000/400 分钟）通常够用；别自建 |
| 5–30 人 | GHA Team 或 GitLab Premium；性能敏感选 CircleCI Performance | $100–600 | 见 §5.6 测算；重点看分钟/ credits 消耗增速 |
| 30–200 人 | GitLab Premium/Ultimate 或 GHA Enterprise + 自托管 runner 混合 | $1,000–5,000 | CDF 2024：混合形态 DORA 部署表现最佳 [cite:50]；开始需要配额治理与缓存策略 |
| 200+ 人 / 多 BU | GitLab self-managed（Ultimate）或 Jenkins（存量）/ Tekton（平台化）+ Argo CD | 定制 | 平台工程化：模板治理、计费分摊（chargeback）、SLA；评估厂商企业档折扣 |

### 8.3 迁移与共存策略

- **共存是常态**：32% 的组织同时使用 2 个 CI 工具、9% 用 3 个以上 [cite:46]——不必追求"一刀切"，可以"新项目上 GHA/GitLab CI，存量系统留守 Jenkins"渐进演进。
- **迁移成本高发点**：流水线逻辑翻译（Groovy→YAML 是重写而非转换）、凭据/制品库迁移、自托管 runner 上的环境定制（特殊编译链、 licensed 工具）、团队肌肉记忆。
- **锁定风险评估**：GHA/GitLab CI 的 YAML 与平台 API 深度耦合，迁移成本随流水线数量线性增长；Jenkins/Tekton 自建方案锁定在基础设施而非厂商。对"厂商定价反复"敏感的团队（GitLab 2020/2022 收紧、GHA 2025 收费未遂均有前科 [cite:25][cite:9]），保持流水线逻辑薄、环境定义厚（容器镜像化）是最通用的对冲。

### 8.4 风险提示与数据时效

- 本报告全部定价/限额核于 2026-10-14；CI/CD 是调价高频领域（一年内即有 GHA 降价、GitLab Credits 两件事），**超过 6 个月请按附录来源重新核价**。
- 推断性结论（锁定成本、趋势外推、GHA self-hosted 远期收费可能）均已标注置信度，决策时请以最新官方公告为准。

---

## 9. 附录：来源清单（访问日期均为 2026-10-14）

**GitHub Actions**
- [cite:1] GitHub Actions billing — https://docs.github.com/en/billing/concepts/product-billing/github-actions
- [cite:2] Actions runner pricing — https://docs.github.com/en/billing/reference/actions-runner-pricing
- [cite:3] Actions limits — https://docs.github.com/en/actions/reference/limits
- [cite:4] GitHub-hosted runners reference — https://docs.github.com/en/actions/reference/runners/github-hosted-runners
- [cite:5] Self-hosted runners reference — https://docs.github.com/en/actions/reference/runners/self-hosted-runners
- [cite:6] Security hardening for GitHub Actions — https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- [cite:7] Update to GitHub Actions pricing（Changelog，2025-12-16） — https://github.blog/changelog/2025-12-16-coming-soon-simpler-pricing-and-a-better-experience-for-github-actions/
- [cite:8] Pricing changes for GitHub Actions（官方说明页） — https://resources.github.com/actions/2026-pricing-changes-for-github-actions/
- [cite:9] Tenki: GitHub Actions Runner Pricing Postponed — https://www.tenki.cloud/blog/github-actions-runner-pricing-2026
- [cite:10] actions/cache README — https://github.com/actions/cache
- [cite:11] About custom actions — https://docs.github.com/en/actions/sharing-automations/creating-actions/about-custom-actions
- [cite:12] How GitHub uses GitHub Actions and larger runners（2023-09） — https://github.blog/2023-09-26-how-github-uses-github-actions-and-actions-larger-runners-to-build-and-test-github-com/
- [cite:13] What is GitHub Actions?（Marketplace 10,000+ 口径） — https://resources.github.com/devops/tools/automation/actions/
- [cite:14] FAQ about changes to GitHub's plans（Team $4 / 3,000 分钟） — https://docs.github.com/en/get-started/learning-about-github/faq-about-changes-to-githubs-plans
- [cite:15] GitHub 定价页（Team $4 / Enterprise $21） — https://github.co.jp/pricing.html
- [cite:16] GitHub Octoverse 2025 — https://github.blog/news-insights/octoverse/octoverse-a-new-developer-joins-github-every-second-as-ai-leads-typescript-to-1/

**GitLab CI**
- [cite:17] Compute minutes | GitLab Docs — https://docs.gitlab.com/ci/pipelines/compute_minutes/
- [cite:18] Executors | GitLab Docs — https://docs.gitlab.com/runner/executors/
- [cite:19] Hosted runners on Linux | GitLab Docs — https://docs.gitlab.com/ci/runners/hosted_runners/linux/
- [cite:20] GitLab tiers | Handbook — https://handbook.gitlab.com/handbook/marketing/brand-and-product-marketing/product-and-solution-marketing/pricing-tiers/
- [cite:21] Purchase additional compute minutes — https://docs.gitlab.com/subscriptions/gitlab_com/compute_minutes/
- [cite:22] GitLab Q1FY2027 Investor Presentation — https://s204.q4cdn.com/984476563/files/doc_financials/2027/q1/GitLab-Q1FY2027-Overview-Presentation.pdf
- [cite:23] GitLab plans API — https://customers.gitlab.com/gitlab_plans
- [cite:24] GitLab for Open Source Program — https://handbook.gitlab.com/handbook/marketing/developer-relations/programs/open-source-program/
- [cite:25] 2020 免费额度收紧公告（gitaly issue #3109） — https://gitlab.com/gitlab-org/gitaly/-/issues/3109
- [cite:26] 2022 公开项目纳入配额（fdroid/admin issue #223） — https://gitlab.com/fdroid/admin/-/issues/223
- [cite:27] GitLab Q4 FY2023 公告（SEC，Premium $19→$29） — https://www.sec.gov/Archives/edgar/data/1653482/000162828023007704/gitlab-ex99120230131fy23.htm
- [cite:28] GitLab Credits and usage billing — https://docs.gitlab.com/subscriptions/gitlab_credits/
- [cite:29] GitLab.com settings — https://docs.gitlab.com/user/gitlab_com/
- [cite:30] Speed up job execution（分布式缓存） — https://docs.gitlab.com/runner/configuration/speed_up_job_execution/
- [cite:31] Storage usage quotas — https://docs.gitlab.com/ee/user/storage_usage_quotas.html
- [cite:32] Proof Points（KnowBe4 案例） — https://handbook.gitlab.com/handbook/sales/command-of-the-message/proof-points/
- [cite:33] GitLab FY2025 Annual Letter（SEC，CACI 案例） — https://www.sec.gov/Archives/edgar/data/1653482/000162828025021579/arsedgargitlabform10k.pdf
- [cite:34] 哔哩哔哩案例 | 极狐 GitLab — https://gitlab.cn/cases/cmkby1ylp000dsy0jbyuluphe

**Jenkins**
- [cite:35] Jenkins Plugins Index（2,000+） — https://plugins.jenkins.io/
- [cite:36] Jenkins LTS Changelog（2.555.1 需 Java 21/25） — https://www.jenkins.io/changelog-stable/index.html
- [cite:37] Jenkins Java Support Policy（Java 25 → LTS 2.541.1） — https://www.jenkins.io/doc/book/platform-information/support-policy-java/
- [cite:38b] Jenkins LTS 2.541.1 Changelog（2026-01-21） — https://www.jenkins.io/changelog-stable/2.541.1/
- [cite:52] Jenkins 加入 CDF 公告（2019，25 万+ 安装） — https://www.jenkins.io/blog/2019/03/12/cdf-launch/
- [cite:52b] CloudBees CI 能力页（Forrester TEI，厂商委托研究） — https://www.cloudbees.com/capabilities/continuous-integration
- [cite:53] CloudBees CI 第三方价格估算（P3） — https://www.deployhq.com/blog/best-ci-cd-software-top-10-tools-to-know-in-2025
- [cite:57] Jenkins and Docker（eBay 案例） — https://www.jenkins.io/solutions/docker/

**CircleCI**
- [cite:41] CircleCI Pricing — https://circleci.com/pricing/
- [cite:43] CircleCI Server 4.10 Overview — https://circleci.com/docs/server-admin/latest/overview/circleci-server-overview/
- [cite:44] CircleCI Server 4.10 Release Notes — https://circleci.com/docs/server-admin/latest/overview/release-notes/
- [cite:48] CircleCI Newsroom（200 万用户、Gartner 2025 Niche） — https://circleci.com/newsroom/circleci-named-a-contender-in-2025-devops-platforms-report/
- [cite:58] CircleCI Jan 4, 2023 security incident report — https://circleci.com/blog/jan-4-2023-incident-report/
- [cite:59] Kajabi 案例 — https://circleci.com/case-studies/kajabi/
- [cite:60] Axios 案例 — https://circleci.com/case-studies/axios/
- [cite:61] ANA Systems 案例 — https://circleci.com/case-studies/ana-systems/

**Tekton 与云原生**
- [cite:38a] Tekton Docs – Pipelines 概念 — https://tekton.dev/docs/pipelines/
- [cite:39] CDF: Tekton Moves to the CNCF（2026-03-24） — https://cd.foundation/announcement/2026/03/24/tekton-moves-to-the-cncf/
- [cite:40] Tekton Pipelines v1.12.0 LTS（2026-05） — https://tekton.dev/blog/2026/05/04/tekton-pipelines-v1.12-0-lts-notifications-controllers-security-hardening-and-performance/
- [cite:42] Red Hat Docs: Understanding OpenShift Pipelines — https://docs.redhat.com/en/documentation/openshift_container_platform/4.5/htmlsingle/pipelines/understanding-openshift-pipelines
- [cite:62] Alauda DevOps Pipelines Docs（Tekton 复杂度评价） — https://docs.alauda.io/alauda-devops-pipelines/4.3/alauda-devops-pipelines-en.pdf
- [cite:63] CNCF: Argo has Graduated（2022-12） — https://www.cncf.io/announcements/2022/12/06/the-cloud-native-computing-foundation-announces-argo-has-graduated/
- [cite:64] CNCF End User Survey（Argo CD 多数采用，2025-07） — https://www.cncf.io/announcements/2025/07/24/cncf-end-user-survey-finds-argo-cd-as-majority-adopted-gitops-solution-for-kubernetes/

**市场与调查数据**
- [cite:45] JetBrains: Best CI/CD Tools for 2026（Ecosystem 2025 组织采用率） — https://blog.jetbrains.com/teamcity/2026/03/best-ci-tools/
- [cite:46] JetBrains: The State of CI/CD in 2025（n=805） — https://blog.jetbrains.com/teamcity/2025/10/the-state-of-cicd/
- [cite:47] GitLab Q4 FY2026 Prepared Remarks — https://s204.q4cdn.com/984476563/files/doc_financials/2026/q4/Q4-FY2026-Earnings-Prepared-Remarks.pdf
- [cite:49] Stack Overflow 2025 Methodology（n=49,009，无 CI 题项） — https://survey.stackoverflow.co/2025/methodology
- [cite:50] CDF: State of CI/CD Report 2024（PDF） — https://cd.foundation/wp-content/uploads/sites/35/2024/04/State-of-CICD-Report-April-22-2024-Updated.pdf
- [cite:51] JetBrains DevEcosystem 2022 – Team Tools — https://www.jetbrains.com/lp/devecosystem-2022/team-tools/

**其他参照工具**
- [cite:54] Azure DevOps Services Pricing — https://azure.microsoft.com/en-us/pricing/details/devops/azure-devops-services/
- [cite:55] Bitbucket Pricing — https://www.atlassian.com/software/bitbucket/pricing
- [cite:56] Buildkite Pricing — https://buildkite.com/pricing
- [cite:65] Woodpecker CI About（Drone 许可变更） — https://woodpecker-ci.org/about
- [cite:66] Travis CI 现状（Idera 收购） — https://sota.io/blog/travis-ci-eu-alternative-2026-idera-delaware-cloud-act-gdpr-ci-cd
