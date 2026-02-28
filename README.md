# Java CI/CD 标准基础模板

一个开箱即用的Java项目CI/CD标准模板，基于GitLab CI/CD设计，支持Maven构建、Docker镜像构建和Kubernetes部署。

## 📁 目录结构

```
java-cicd-template/
├── .gitlab-ci.yml              # 主CI配置文件
├── ci/
│   ├── templates/              # CI/CD模板定义
│   │   ├── common.yml          # 通用配置模板
│   │   ├── maven.yml           # Maven构建模板
│   │   ├── docker.yml          # Docker镜像构建模板
│   │   └── k8s.yml             # Kubernetes部署模板
│   ├── docker/
│   │   └── Dockerfile          # 标准Dockerfile
│   └── scripts/                # 脚本目录 (预留)
├── example/
│   └── .gitlab-ci.yml          # 示例子项目配置
└── README.md                   # 本文档
```

## 🚀 快速开始

### 方式一: 作为子项目引入

在其他Java项目中创建 `.gitlab-ci.yml`:

```yaml
include:
  - project: 'cidevops/java-cicd-template'
    file: '.gitlab-ci.yml'

variables:
  APP_NAME: my-java-app
  DOCKER_REGISTRY: registry.example.com
  K8S_NAMESPACE: production
```

### 方式二: 复制模板

将本仓库克隆到你的项目中:

```bash
git clone https://gitlab-ui.test.com/cidevops/java-cicd-template.git
# 复制必要文件到你的项目
cp -r ci/ your-project/
cp .gitlab-ci.yml your-project/
```

## 📝 配置说明

### 必需变量

| 变量名 | 说明 | 示例 |
|--------|------|------|
| `APP_NAME` | 应用名称 | `my-java-app` |
| `DOCKER_REGISTRY` | Docker镜像仓库地址 | `registry.example.com` |
| `K8S_NAMESPACE` | Kubernetes命名空间 | `production` |

### 可选变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `DOCKERFILE_PATH` | 自定义Dockerfile路径 | `ci/docker/Dockerfile` |
| `MAVEN_SETTINGS` | Maven settings.xml (Base64) | 内置默认配置 |
| `NEXUS_USERNAME` | Nexus仓库用户名 | - |
| `NEXUS_PASSWORD` | Nexus仓库密码 | - |

## 🔄 流水线阶段

| 阶段 | 说明 | 触发条件 |
|------|------|----------|
| **build** | Maven编译打包 | 所有分支和标签 |
| **test** | 单元测试 | 所有分支和标签 |
| **docker** | 构建并推送Docker镜像 | 所有分支和标签 |
| **deploy** | 部署到Kubernetes | 手动触发 |

## 🌿 分支策略

| 分支 | 构建 | 测试 | Docker | 部署 |
|------|------|------|--------|------|
| `develop` | ✅ | ✅ | ✅ | 手动(开发环境) |
| `feature/*` | ✅ | ✅ | ✅ | - |
| `master/main` | ✅ | ✅ | ✅ | 手动(生产环境) |
| `tag` | ✅ | ✅ | ✅ | 手动(生产环境) |

## 📦 Docker镜像标签策略

| 触发源 | 镜像标签 |
|--------|----------|
| Tag (如 v1.0.0) | `v1.0.0`, `latest` |
| master/main 分支 | `latest` |
| 其他分支 | Git commit short SHA |

## ☸️ Kubernetes部署

### 环境映射

- **开发环境**: `develop` 或 `dev` 分支 → `K8S_NAMESPACE-dev`
- **测试环境**: `develop` 分支 → `K8S_NAMESPACE-test`
- **生产环境**: `master`/`main` 分支或 Tag → `K8S_NAMESPACE`

### 自定义K8s配置

你可以在项目中创建 `k8s/{env}/deployment.yml` 来覆盖默认部署配置:

```
your-project/
├── k8s/
│   ├── dev/
│   │   └── deployment.yml
│   ├── test/
│   │   └── deployment.yml
│   └── prod/
│       └── deployment.yml
└── .gitlab-ci.yml
```

### 回滚操作

每个环境都支持一键回滚到上一版本:
- 在GitLab UI中点击对应环境的 "Rollback" 按钮

## 🔧 高级配置

### 自定义Maven构建命令

```yaml
maven_build:
  script:
    - mvn clean compile -B -U -Pcustom-profile
```

### 自定义测试覆盖

```yaml
maven_test:
  script:
    - mvn test -B -Pintegration-tests
  coverage: '/Total.*?([0-9]{1,3})%/'
```

### 自定义部署脚本

```yaml
deploy_prod:
  script:
    - echo "执行自定义部署逻辑"
    - kubectl set image deployment/${APP_NAME} ...
```

## 📋 GitLab Runner 要求

确保你的GitLab Runner满足以下要求:

1. **Maven构建**: 需要安装Maven或使用Docker镜像
2. **Docker构建**: 需要配置Docker-in-Docker (dind)
3. **Kubernetes部署**: 需要配置Kubeconfig

推荐配置 `.gitlab-runner/config.toml`:

```toml
[[runners]]
  name = "docker-runner"
  executor = "docker"
  [runners.docker]
    image = "ubuntu:22.04"
    volumes = ["/cache"]
    privileged = true  # 需要用于Docker-in-Docker
```

## 📄 许可证

MIT License
