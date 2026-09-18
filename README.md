# Java CI/CD 标准基础模板

一个开箱即用的 Java 项目 CI/CD 标准模板，基于 GitLab CI/CD 设计，支持 Maven 构建、Docker 镜像构建和 Kubernetes 部署。

## 📁 目录结构

```
java-cicd-template/
├── jobs/                                # CI/CD Job 模板定义
│   ├── common_java/                     # 单体 Java 项目 (4 阶段)
│   │   ├── package.yaml                 #   Maven 构建
│   │   ├── test.yaml                    #   单元测试 + JaCoCo
│   │   ├── build.yaml                   #   Docker 构建 + 推送
│   │   ├── deploy.yaml                  #   K8s 部署 (多环境分支)
│   │   └── scan.yaml                    #   Trivy 镜像扫描
│   ├── common_aggregate/                # 聚合项目 (4 阶段, 新增 v1.0)
│   │   ├── package.yaml                 #   Maven 构建 (矩阵, 带 -am 依赖)
│   │   ├── test.yaml                    #   单元测试 (矩阵)
│   │   ├── build.yaml                   #   Docker 构建 + 推送 (矩阵)
│   │   ├── deploy.yaml                  #   K8s 部署 (矩阵 + 多环境分支)
│   │   └── scan.yaml                    #   Trivy 镜像扫描 (矩阵)
│   ├── common_vue/                      # Vue 前端项目
│   └── debug-cert.yaml                  # 证书调试
├── templates/                           # 流水线模板
│   ├── java-pipeline.yaml               # 单体项目入口 (向后兼容)
│   ├── java-pipeline-aggregate.yaml     # 聚合项目入口 (v1.0)
│   ├── common-java-pipeline.yaml        # 单体公共入口
│   └── common-vue-pipeline.yaml         # Vue 入口
└── README.md
```

## 🚀 快速开始

### 方式一: 作为子项目引入

在其他 Java 项目中创建 `.gitlab-ci.yml`:

```yaml
include:
  - project: 'cidevops/java-cicd-template'
    ref: master
    file: 'templates/java-pipeline.yaml'

variables:
  # 应用名称
  APP_NAME: my-java-app
  # Docker Registry 地址
  CI_REGISTRY: registry.example.com
  # Kubernetes 命名空间
  K8S_NAMESPACE: production
```

### 方式二: 复制模板

将本仓库克隆到你的项目中:

```bash
git clone https://gitlab-ui.test.com/cidevops/java-cicd-template.git
# 复制必要文件到你的项目
cp -r jobs/ your-project/
cp -r templates/ your-project/
```

## 📝 配置说明

### 统一变量 (templates/java-pipeline.yaml)

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `MAVEN_IMAGE` | Maven 构建镜像 | `maven:3.9.4-eclipse-temurin-17` |
| `DOCKER_IMAGE` | Docker 构建镜像 | `docker:cli` |
| `KUBECTL_IMAGE` | Kubernetes 客户端镜像 | `bitnami/kubectl:latest` |
| `SKYWALKING_AGENT_IMAGE` | SkyWalking 探针镜像 | `apache/skywalking-java-agent:9.6.0-java17` |

### 构建相关变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `BUILD_SHELL` | Maven 构建命令 | `mvn clean package -B -DskipTests -f pom.xml -s settings.xml` |
| `TEST_SHELL` | 测试命令 | `mvn test -B -U -f pom.xml -s settings.xml` |
| `ARTIFACTS` | 构建产物路径 | 项目指定的 target 目录 |
| `CACHE_DIR` | 缓存目录 | 项目指定的 target 目录 |
| `JUNIT_REPORT_PATH` | 测试报告路径 | `target/surefire-reports/` |

### Docker 相关变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `CI_REGISTRY` | Docker Registry 地址 | `k8s-node-1:5000` |
| `CI_REGISTRY_IMAGE` | 镜像名称 | 项目指定 |
| `DOCKERFILE_PATH` | Dockerfile 路径 | 项目指定 |

### Kubernetes 部署变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `APP_NAME` | 应用名称 | 项目指定 |
| `K8S_NAMESPACE` | 命名空间 | `default` |
| `K8S_YAML` | K8s 部署文件路径 | 项目指定 |

### 内置 CI/CD 变量

GitLab 内置变量可直接使用:

| 变量名 | 说明 |
|--------|------|
| `CI_COMMIT_SHA` | 当前提交的 SHA-1 哈希值 |
| `CI_COMMIT_SHORT_SHA` | 当前提交的短 SHA |
| `CI_COMMIT_TAG` | 当前提交的标签名称 |
| `CI_COMMIT_BRANCH` | 当前提交的分支名称 |
| `CI_COMMIT_REF_NAME` | 当前提交的分支或标签名称 |

## 🔒 变量契约（common_java ↔ java-pipeline-harbor）

> **目标**：两个单体家族的**同名变量同值 → 产物一致**；确需特殊值时必须登记于本表，不得私下改。
> **变量优先级**（GitLab 官方）：项目级 CI/CD 变量 > job 级变量 > 顶层/default 变量（含 include 的模板 `variables`）。

| # | 变量 | `common-java-pipeline.yaml` | `java-pipeline-harbor.yaml` | 处置 | 说明 |
|---|------|------------------------------|------------------------------|------|------|
| 1 | `MAVEN_IMAGE` | `maven:3.9.6-eclipse-temurin-17` | 同左 | 统一 | Maven 版本影响 jar |
| 2 | `BUILD_SHELL` | `-s /etc/maven/settings.xml` | `-s settings.xml` | **暂挂待议** | 内网 Nexus3 vs 项目自带(公网) |
| 3 | `TEST_SHELL` | 同 #2 | 同 #2 | **暂挂待议** | 同上 |
| 4 | `DOCKER_IMAGE` | `docker:cli` | `docker:24`（job 级 `DOCKER_BUILDKIT=0`） | Harbor 专有 | classic builder；Docker 25+ 已移除 |
| 5 | `KUBECTL_IMAGE` | `bitnami/kubectl:latest` | `docker.io/bitnamilegacy/kubectl:1.30.3` | 各自保留 | 公网 `bitnami/kubectl:1.30.x` tag 已 404 |
| 6 | `DOCKERFILE_PATH` | `src/main/docker/Dockerfile` | 同左 | 统一 | 全项目目录约定 |
| 7 | `CI_REGISTRY` | `k8s-node-1:5000` | `harbor-ui.test.com` | 各自保留 | 环境绑定 |
| 8 | `IMAGE_PULL_SECRETS` | `docker-secret` | `harbor-secret` | 各自保留 | 由 deploy job 创建 |
| 9 | `CI_REGISTRY_USER` / `CI_REGISTRY_PASSWORD` | 注释（走 `DOCKER_AUTH_CONFIG`） | 注释 | 项目级注入 | 模板不设默认值 |
| 10 | `SKYWALKING_AGENT_IMAGE` | `apache/skywalking-java-agent:9.6.0-java17` | 同左 | 统一 | |
| 11 | `CI_DEBUG_TRACE` | `"false"` | 同左 | 统一 | |
| 12 | `GIT_CHECKOUT` | `"true"` | 同左 | 统一 | |
| 13 | `CI_REGISTRY_FULL_IMAGE` | `$CI_REGISTRY/$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA` | 同左 | 统一 | 仅作默认值，job 内会重算 |
| 14 | `DOCKER_CONFIG` | 内置 base64（`k8s-node-1:5000`） | — | common_java 专有 | 非-Harbor 的 imagePullSecret 内容 |
| 15 | `JACOCO_REPORT_DIR` | `target/site/jacoco/` | 同左 | 统一 | test 上传产物集合一致 |
| 16 | `TRIVY_IMAGE` | — | `$CI_REGISTRY/library/trivy:latest` | Harbor 专有 | 内网离线，用 Harbor 上的 trivy |

### 其余 [通用] 同名同值变量

`MAVEN_OPTS`、`CACHE_DIR` / `ARTIFACTS` / `ARTIFACTS_NAME`、`JUNIT_REPORT_PATH` / `JACOCO_REPORT_PATH`、
`APP_NAME`、`K8S_BASE_PATH` / `K8S_FILE` / `K8S_ENV` / `K8S_NAMESPACE`、
`SERVICE_TYPE` / `SERVICE_PORT` / `CONTAINER_PORT` / `IMAGE_PULL_POLICY`、
`REPLICAS` / `MAX_SURGE` / `MAX_UNAVAILABLE`、
`MEMORY_REQUEST` / `CPU_REQUEST` / `MEMORY_LIMIT` / `CPU_LIMIT`、
`HEALTH_CHECK_PATH` / `LIVENESS_INITIAL_DELAY` / `READINESS_INITIAL_DELAY`、
`JAVA_MIN_HEAP` / `JAVA_MAX_HEAP`、`HPA_MIN_REPLICAS` / `HPA_MAX_REPLICAS` / `HPA_CPU_THRESHOLD` / `HPA_MEMORY_THRESHOLD` / `HPA_SCALE_DOWN_WINDOW`

### 凭据注入（统一口径）

`CI_REGISTRY_USER` / `CI_REGISTRY_PASSWORD` 一律由**项目级 CI/CD 变量**注入：
- 非-Harbor：推荐 `DOCKER_AUTH_CONFIG`（GitLab 自动注入 `~/.docker/config.json`）
- Harbor：`CI_REGISTRY_USER` / `CI_REGISTRY_PASSWORD` 供 job 内 `docker login` 使用

## 🔄 流水线阶段

### 单体项目 (java-pipeline.yaml)

| 阶段 | 说明 | Job 名称 | 触发条件 |
|------|------|----------|----------|
| **debug-cert** | 证书调试 | `debug-cert` | 所有分支和标签 |
| **package** | Maven 编译打包 | `mvn_build` | 所有分支和标签 |
| **test** | 单元测试 + 覆盖率 | `test` | 所有分支和标签 |
| **build** | Docker 镜像构建并推送 | `docker_upload` | master 分支或 tags |
| **deploy** | Kubernetes 部署 | `k8s_deploy` | master 分支 (需上游 job 完成) |

### 聚合项目 (java-pipeline-aggregate.yaml)

| 阶段 | 说明 | Job 名称 | 触发条件 |
|------|------|----------|----------|
| **validate** | 全项目编译验证 | `aggregate:compile` | 所有分支和 MR |
| **test** | 单元测试 (matrix) | `aggregate:test` × N | 所有分支和 MR |
| **build** | Docker 构建 (matrix) | `aggregate:build` × N | main 分支 / tag + rules:changes |
| **deploy** | K8s 部署 (matrix) | `aggregate:deploy` × N | tag 自动 / main 手动 |

## 📦 Job 模板详解

### jobs/package.yaml - Maven 构建(单体)

- **模板名称**: `.mvn_build`
- **功能**: Maven 编译打包，包含缓存优化和自动重试
- **产物**: JAR 文件保存 30 天
- **缓存**: Maven 本地仓库 `.m2/repository`

### jobs/test.yaml - 测试(单体)

- **模板名称**: `.test`
- **功能**: 执行单元测试和 JaCoCo 覆盖率统计
- **报告**: JUnit XML 报告 + JaCoCo 覆盖率报告

### jobs/build.yaml - Docker 构建(单体)

- **模板名称**: `.docker_build_base` / `.docker_upload`
- **功能**: 构建并推送 Docker 镜像到 Registry
- **特性**: 自动重试、缓存优化、多标签支持

### jobs/deploy.yaml - K8s 部署(单体)

- **模板名称**: `.k8s_deploy`
- **功能**: 部署应用到 Kubernetes 集群
- **特性**: 自动 namespace 创建、部署验证、回滚支持

---

## 🆕 聚合项目支持 (v1.0+)

### 适用场景

聚合项目(微服务架构)与单体项目有本质区别:
- **单体**: 1 个 APP_NAME、1 个 K8s YAML、1 个镜像
- **聚合**: N 个可部署应用(8+)、N 个 K8s YAMLs、N 个镜像

### 特性

| 特性 | 说明 |
|------|------|
| **parallel:matrix** | 8 个应用并行构建/部署 |
| **rules:changes** | 基础库变更自动触发依赖应用,只构建变更部分 |
| **统一部署** | 8 个独立 K8s YAML,共享模板只差镜像 tag |
| **复用单体变量** | 镜像/标签策略完全一致,降低学习成本 |

### 快速开始

在 SpringBootLearning 类聚合项目 `.gitlab-ci.yml`:

```yaml
include:
  - project: 'cidevops/java-cicd-template'
    ref: master
    file: 'templates/java-pipeline-aggregate.yaml'

variables:
  # 镜像仓库
  CI_REGISTRY: harbor-ui.master.com
  CI_REGISTRY_IMAGE: springboot        # 项目名
  CI_REGISTRY_USER: $HARBOR_ROBOT_USER
  CI_REGISTRY_PASSWORD: $HARBOR_ROBOT_PASSWORD

  # 部署
  K8S_NAMESPACE: app-pre-prod
  KUBECONFIG_CONTENT: $KUBECONFIG_CONTENT
```

**注意**:模板内置了 SpringBootStarry 8 个微服务的 matrix 配置。如果你的项目结构不同,需要 fork 模板并修改 `templates/java-pipeline-aggregate.yaml` 中的 matrix。

### 流水线架构

```
validate (compile 全项目)
    ↓
matrix.test (8 个并行,各 app 单独测试)
    ↓
matrix.build (8 个并行,基于 rules:changes 增量)
    ↓
matrix.deploy (8 个并行,tag 触发或手动)
```

### Job 模板详解 (aggregate.yaml)

| 模板名 | 功能 | 关键变量 |
|--------|------|----------|
| `.aggregate_maven_compile` | 全项目编译验证 | - |
| `.aggregate_mvn_build` | 单 app Maven 构建,带 `-am` 依赖 | `APP_NAME`, `APP_MAVEN_PATH` |
| `.aggregate_test` | 单 app 单元测试 + JaCoCo | `APP_NAME`, `APP_MAVEN_PATH` |
| `.aggregate_docker_build` | 单 app Docker 构建 + push Harbor | `APP_NAME`, `APP_MAVEN_PATH`, `IMAGE_TAG` |
| `.aggregate_k8s_deploy` | 单 app K8s 部署(`kubectl set image`) | `APP_NAME`, `K8S_NAMESPACE` |

### 镜像命名规则

聚合项目下每个 app 镜像命名格式:
```
$CI_REGISTRY/$CI_REGISTRY_IMAGE/$APP_NAME:$IMAGE_TAG
```

例如:`harbor-ui.master.com/springboot/starry-gateway:v1.0.0`

### 部署文件组织

每个 app 需在项目仓库准备独立的 K8s YAML:
```
项目根/
├── k8s/
│   ├── starry-gateway.yaml         # APP_NAME=starry-gateway
│   ├── starry-datacenter.yaml      # APP_NAME=starry-datacenter
│   ├── starry-admin.yaml           # APP_NAME=starry-admin
│   ├── starry-api.yaml             # APP_NAME=starry-api
│   ├── starry-auth.yaml            # APP_NAME=starry-auth
│   ├── starry-oauth-server.yaml    # APP_NAME=starry-oauth-server
│   ├── starry-oauth-resource.yaml  # APP_NAME=starry-oauth-resource
│   └── starry-sentinel.yaml        # APP_NAME=starry-sentinel
└── SpringBootStarry/
    ├── StarryGateway/
    │   └── src/main/docker/Dockerfile   # Dockerfile 路径约定
    └── ...
```

### 触发逻辑

| 触发源 | 行为 |
|--------|------|
| MR / feature 分支 | 仅 validate + test(不构建镜像) |
| main push | validate + test + build(增量构建变更的 app)|
| tag (v*.*.*) | validate + test + build + deploy(全部构建并部署)|

### 增量触发规则

构建阶段 (`aggregate:build`) 使用 `rules:changes`:
- 基础库路径变更(`StarryCommon/**` 等)→ **所有 app 重建**
- 单 app 源码变更 → 仅该 app 重建
- 仅文档/无关文件 → 不触发构建

### 自定义矩阵

如果项目结构与 SpringBootStarry 不同,需 fork 模板并修改 matrix:

```yaml
# templates/java-pipeline-aggregate.yaml (自定义版本)
aggregate:test:
  extends: .aggregate_test
  parallel: 4
  matrix:
    - APP_NAME: "my-service-a"
      APP_MAVEN_PATH: "services/service-a"
    - APP_NAME: "my-service-b"
      APP_MAVEN_PATH: "services/service-b"
    # 添加你的应用...
```

## 🌿 分支策略

| 分支 | 构建 | 测试 | Docker 镜像 | K8s 部署 |
|------|------|------|-------------|----------|
| `feature/*` | ✅ | ✅ | ✅ | - |
| `develop` | ✅ | ✅ | ✅ | 手动 |
| `master` | ✅ | ✅ | ✅ | 手动 |
| `tag` | ✅ | ✅ | ✅ | 手动 |

## 🏷️ Docker 镜像标签策略

| 触发源 | 镜像标签 |
|--------|----------|
| Tag (如 v1.0.0) | `v1.0.0` |
| master/main 分支 | `latest` |
| 其他分支 | Git commit short SHA |

## ☸️ Kubernetes 部署

### 前置要求

1. 配置 `KUBECONFIG_CONTENT` CI/CD 变量（Base64 编码的 kubeconfig）: cat ~/.kube/config | base64 -w 0 或者使用原始的值，具体看deploy.yaml
2. 确保目标集群可访问
3. 准备好 K8s 部署 YAML 文件

### 部署流程

1. **构建阶段**: Maven 编译打包 → 生成 JAR
2. **测试阶段**: 执行单元测试 → 生成测试报告
3. **镜像阶段**: 构建 Docker 镜像 → 推送到 Registry
4. **部署阶段**: 更新 K8s Deployment 镜像 → 验证部署状态

### 回滚操作

```bash
# 使用 kubectl 回滚到上一版本
kubectl rollout undo deployment/${APP_NAME} -n ${K8S_NAMESPACE}

# 查看部署状态
kubectl rollout status deployment/${APP_NAME} -n ${K8S_NAMESPACE}
```

## 🔧 高级配置

### 自定义 Maven 构建命令

```yaml
variables:
  BUILD_SHELL: "mvn clean package -B -U -Pcustom-profile -f pom.xml"
```

### 自定义测试配置

```yaml
variables:
  TEST_SHELL: "mvn test -B -Pintegration-tests -f pom.xml"
```

### 自定义 Docker 镜像名称

```yaml
variables:
  CI_REGISTRY: "my-registry.example.com"
  CI_REGISTRY_IMAGE: "my-app"
```

### 使用私有 Maven 仓库

在 `settings.xml` 中配置私有仓库，并在变量中指定:

```yaml
variables:
  MAVEN_SETTINGS: "your-base64-encoded-settings.xml"
```

## 📋 GitLab Runner 要求

确保你的 GitLab Runner 满足以下要求:

### Tag 配置

| Tag 名称 | 用途 | 说明 |
|----------|------|------|
| `package` | Maven 构建 | 需要 Maven 镜像 |
| `test` | 测试执行 | 需要 Maven 镜像 |
| `build` | Docker 构建 | 需要 Docker 镜像 |
| `deploy` | K8s 部署 | 需要 kubectl 镜像 |

### 推荐 Runner 配置

```toml
[[runners]]
  name = "java-cicd-runner"
  executor = "docker"
  [runners.docker]
    image = "maven:3.9.4-eclipse-temurin-17"
    privileged = true  # 需要用于 Docker-in-Docker
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
```

## 📄 许可证

MIT License
