# Java CI/CD 标准基础模板

一个开箱即用的 Java 项目 CI/CD 标准模板，基于 GitLab CI/CD 设计，支持 Maven 构建、Docker 镜像构建和 Kubernetes 部署。

## 📁 目录结构

```
java-cicd-template/
├── jobs/                           # CI/CD Job 模板定义
│   ├── build.yaml                  # Docker 镜像构建模板
│   ├── deploy.yaml                 # Kubernetes 部署模板
│   ├── package.yaml                # Maven 构建打包模板
│   └── test.yaml                   # Java 测试模板
├── templates/                      # 流水线模板
│   └── java-pipeline.yaml          # 主 CI/CD 流水线配置
└── README.md                       # 本文档
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

## 🔄 流水线阶段

| 阶段 | 说明 | Job 名称 | 触发条件 |
|------|------|----------|----------|
| **package** | Maven 编译打包 | `mvn_build` | 所有分支和标签 |
| **test** | 单元测试 + 覆盖率 | `test` | 所有分支和标签 |
| **build** | Docker 镜像构建并推送 | `docker_upload` | master 分支或 tags |
| **deploy** | Kubernetes 部署 | `k8s_deploy` | master 分支 (需上游 job 完成) |

## 📦 Job 模板详解

### jobs/package.yaml - Maven 构建

- **模板名称**: `.mvn_build`
- **功能**: Maven 编译打包，包含缓存优化和自动重试
- **产物**: JAR 文件保存 30 天
- **缓存**: Maven 本地仓库 `.m2/repository`

### jobs/test.yaml - 测试

- **模板名称**: `.test`
- **功能**: 执行单元测试和 JaCoCo 覆盖率统计
- **报告**: JUnit XML 报告 + JaCoCo 覆盖率报告

### jobs/build.yaml - Docker 构建

- **模板名称**: `.docker_build_base` / `.docker_upload`
- **功能**: 构建并推送 Docker 镜像到 Registry
- **特性**: 自动重试、缓存优化、多标签支持

### jobs/deploy.yaml - K8s 部署

- **模板名称**: `.k8s_deploy`
- **功能**: 部署应用到 Kubernetes 集群
- **特性**: 自动 namespace 创建、部署验证、回滚支持

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
