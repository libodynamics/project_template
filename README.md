<!-- Copyright The Project Template Contributors -->

# 项目名称

> **模板使用说明**
>
> 本仓库是公司项目模板。使用模板创建新项目后，必须把本文件中的项目名称、说明、环境要求和所有 `TODO` 命令替换成项目真实内容；不适用的章节应删除或改写。
>
> 维护 `project_template` 仓库时，本文件中的 `TODO` 是模板占位内容。除非本次变更目标就是调整模板占位策略，否则不要把这些占位内容替换成模板仓库自身信息。

TODO：用一段话说明本项目是什么。

## 模板仓库维护

维护 `project_template` 仓库时，修改模板结构或规则后至少执行：

```bash
docker build --pull -f .devcontainer/base.Dockerfile -t ghcr.io/libodynamics/project_template/devcontainer:latest .devcontainer
docker build --pull=false -f .devcontainer/Dockerfile -t project-template-devcontainer:latest .devcontainer
DEVCONTAINER_USER="$(id -un | sed -E 's/[^[:alnum:]_.-]+/-/g; s/^-+//; s/-+$//')"
DEVCONTAINER_BRANCH="$(git branch --show-current | sed -E 's/[^[:alnum:]_.-]+/-/g; s/^-+//; s/-+$//')"
if [ -z "$DEVCONTAINER_BRANCH" ]; then echo "detached HEAD is not allowed for the devcontainer name" >&2; exit 1; fi
export DEVCONTAINER_NAME="project-template-devcontainer-${DEVCONTAINER_USER}-${DEVCONTAINER_BRANCH}"
docker inspect "$DEVCONTAINER_NAME" >/dev/null 2>&1 || docker run -d --name "$DEVCONTAINER_NAME" -v "$PWD:/workspace" -w /workspace project-template-devcontainer:latest sleep infinity
if [ "$(docker inspect -f '{{.State.Running}}' "$DEVCONTAINER_NAME")" != "true" ]; then docker start "$DEVCONTAINER_NAME" >/dev/null; fi
docker exec "$DEVCONTAINER_NAME" bash -lc 'git config --global --add safe.directory /workspace && git status --short --branch && rg -n "TODO|YYYY-MM-DD|@TODO-owner" . && pre-commit run --all-files && rustc --version && node --version && npm --version && devcontainer --version && mmdc --version && plantuml -version && ncu --version'
```

`TODO` 命中应来自模板占位或示例。新增、删除或移动模板文件时，应同步检查 `README.md`、`AGENTS.md`、`docs/README.md` 和 `docs/conventions.md` 中的文档入口。

上述流程中，宿主机只负责镜像构建、计算容器名、创建或启动常驻 Dev Container；`git status`、`rg`、pre-commit 和工具版本检查都必须通过 `docker exec` 在容器内执行。需要重建同名容器时，应先确认旧容器没有需要保留的状态，再停止并删除后重建。更完整的容器说明见 `.devcontainer/README.md`。

## 派生项目初始化 checklist

使用本模板创建新项目后，至少完成以下事项：

- [ ] 替换 `README.md`、`AGENTS.md`、`SECURITY.md` 和文档模板中的项目占位内容。
- [ ] 确认项目名称、仓库名、包名、镜像名、Dev Container `name`、常驻 Dev Container 容器名、项目运行时容器名与发布目标。
- [ ] 默认删除 `.devcontainer/base.Dockerfile` 和基础镜像发布 workflow；只有项目要维护自己的基础镜像时才保留，并说明镜像名、发布目标和维护责任。
- [ ] 如果继承的基础镜像是 private GHCR package，在该 package 的 `Manage Actions access` 中授予本仓库 `Read` 权限，并在 CI 中保留 `packages: read` 与 `docker/login-action`。
- [ ] 根据真实团队更新 `.github/CODEOWNERS`。
- [ ] 确认 MIT 许可证是否适用；不适用时替换 `LICENSE` 并同步 README。
- [ ] 补齐安装、编译、运行、测试、打包、发布命令；这些命令必须在常驻 Dev Container 容器或 CI/self-hosted runner 中执行。
- [ ] 声明如何拉起、复用和必要时重建常驻 Dev Container 容器，容器名必须包含项目名、用户名和当前 Git 分支名。
- [ ] 声明 Docker/Dev Container 编译、打包和部署产物目录，确保容器内输出路径和宿主机可见路径匹配，且需要保留的产物写回当前项目目录。
- [ ] 补齐架构边界、运行模式、验证模式、硬件/外部服务依赖和残余风险。
- [ ] 根据项目需要启用或调整 branch protection、DCO、CI、review 和发布门禁。
- [ ] 删除不适用的硬件、生产、供应商、SOP 或文档模板章节。

## 快速开始

TODO：写清楚本项目实际使用的命令。

公司不强制统一命令名。每个项目自己的安装、编译、运行、测试、打包、发布方式都在本文件中说明。
派生项目必须把这些命令补成可直接执行的项目真值源。除 Git 和 Docker/Dev Container 编排命令外，项目命令不直接在宿主机执行，必须先启动常驻 Dev Container，再通过 `docker exec "$DEVCONTAINER_NAME" ...`、`devcontainer exec` 或 CI/self-hosted runner 中的等价容器入口运行。命令可以由 AI agent 根据真实项目生成或维护，但合并前必须说明容器入口、前置条件，以及是否依赖硬件、外部服务或网络。

## 环境要求

- Git
- Docker / Docker Desktop
- 支持 Dev Container 的编辑器或 AI agent
- TODO：需要通过 Docker device、volume、network、self-hosted runner 或 CI 暴露给容器的硬件、签名或外部服务条件

## 开发环境

默认使用 `.devcontainer/`。

项目默认不维护宿主机本地运行路径。安装依赖、编译、运行、测试、生成、打包、发布和 pre-commit hooks 都必须先启动常驻后台的 Dev Container 容器，再通过 `docker exec "$DEVCONTAINER_NAME" ...` 或 CI/self-hosted runner 中的等价容器入口执行；宿主机只负责 Git 和 Docker/Dev Container 编排。

默认 Dev Container 基于 `ghcr.io/libodynamics/project_template/devcontainer:latest`，并安装通用开发、文档、协作、Node.js 和 Rust 基础工具。模板仓库用 `.devcontainer/base.Dockerfile` 构建并发布这个基础镜像；派生项目在 `.devcontainer/Dockerfile` 中继续 `FROM` 该 `latest` 镜像并按需补充 SDK、数据库、模拟器、硬件工具或项目专用依赖。

`.devcontainer/base.Dockerfile` 仅供 `project_template` 仓库维护通用基础镜像。派生项目默认删除它和对应的基础镜像发布 workflow；如果项目需要自建基础镜像，必须在本文件说明镜像名、tag 策略、发布 registry、触发条件、维护责任和回滚方式。

如果基础镜像保持为 private GHCR package，派生项目的 GitHub Actions 需要两个前置条件：先在基础镜像 package 的 `Manage Actions access` 中给派生仓库授予 `Read` 权限，再在 CI job 中声明 `packages: read` 并使用 `docker/login-action` 登录 `ghcr.io`。不要把 PAT 或真实 token 写入仓库；非 GitHub CI 或开发者本机拉取 private 镜像时，应使用具备 `read:packages` scope 的个人或机器人 token，通过 CI secret 或本机 `docker login ghcr.io` 注入。

模板仓库的 Docker 命名约定如下，派生项目应在初始化时替换为自己的稳定名称：

| 用途 | 名称 |
|------|------|
| 基础 Dev Container 镜像 | `ghcr.io/libodynamics/project_template/devcontainer:latest` |
| 本地派生 Dev Container 镜像 | `project-template-devcontainer:latest` |
| Dev Container 显示名 | `project-template-devcontainer` |
| 常驻 Dev Container 容器名 | `project-template-devcontainer-{username}-{branch}` |
| 项目服务/应用运行时容器名 | `project-template-{service}-{username}-{branch}` |

命名规则：

- 常驻 Dev Container 容器名必须使用 `{project}-devcontainer-{username}-{branch}`。
- 项目服务或应用运行时容器名必须使用 `{project}-{service}-{username}-{branch}`。
- 示例：`ranger-devcontainer-nzh-main`、`ranger-daemon-nzh-main`。
- `username` 必须先归一化为 Docker 容器名允许的字符。
- `service` 必须是本文件声明的稳定服务标识，例如 `daemon`、`web`、`db`、`sim`。
- `branch` 必须来自当前具名 Git 分支，例如 `git branch --show-current`。分支名中的 `/`、空格和其他特殊字符必须替换为 `-`。detached HEAD 状态不得启动常驻 Dev Container 或项目运行时容器，必须先切换到具名分支。
- 使用 Dev Container CLI 或编辑器打开项目前，先设置 `DEVCONTAINER_NAME`，并确保它与 `devcontainer.json`、Docker Compose 和手动 `docker run` 示例一致。

TODO：说明如何拉起、复用和必要时重建常驻 Dev Container，以及本项目额外的 `postCreateCommand` 会安装什么。项目命令示例必须先创建或启动常驻容器，再通过 `docker exec "$DEVCONTAINER_NAME" ...` 执行，不得用一次性 `docker run --rm ... <项目命令>` 作为常规入口。

## 运行模式与验证模式

> 创建新项目时，删除不适用的模式，补齐适用模式的命令、运行位置和前置条件。

| 模式 | 用途 | 运行位置 | 是否需要硬件/外部服务 | 命令入口 |
|------|------|----------|------------------------|----------|
| 容器内开发 | TODO | 常驻 Dev Container | TODO | TODO |
| CI | TODO | TODO：GitHub Actions / GitLab CI / self-hosted runner | TODO | TODO |
| 仿真/模拟器 | TODO：例如 QEMU、浏览器、mock 服务 | TODO：常驻 Dev Container / CI | TODO | TODO |
| 无硬件运行 | TODO：验证配置、线程编排或基础流程 | TODO：常驻 Dev Container / CI | 否 | TODO |
| 硬件诊断 | TODO：验证设备、总线、传感器或产线夹具 | TODO | 是 | TODO |
| 生产/部署 | TODO：打包、发布、安装、升级、回滚 | TODO：常驻 Dev Container / self-hosted runner / 产线容器环境 | TODO | TODO |

涉及硬件设计、生产、供应商交付物或 SOP 的项目，应在 `docs/templates/` 对应模板基础上建立正式文档，并在本节链接。

## 安装依赖

以下项目命令默认先启动常驻 Dev Container，再通过 `docker exec "$DEVCONTAINER_NAME" ...`、`devcontainer exec` 或等价的容器入口执行；宿主机侧只保留容器编排入口。

```bash
# TODO
```

## 编译

```bash
# TODO
```

## 运行

```bash
# TODO
```

## 测试

```bash
# TODO
```

## 打包与发布

```bash
# TODO
```

打包或部署产物必须由 Dev Container、Docker Compose、CI 或 self-hosted runner 中的隔离环境生成。本节必须写清楚：

- 宿主机项目目录下的产物目录，例如 `dist/`、`build/`、`target/release/`、`out/` 或项目自定义目录。
- 容器内对应输出路径，例如 `/workspace/dist`，并确保它来自当前项目 bind mount，而不是容器临时目录、匿名 volume、用户主目录或项目外路径。
- 发布命令如何从宿主机可见目录读取产物，以及部署、验收、回滚使用的是哪个路径。
- 需要保留的产物名称、校验方式和清理方式。

## 项目文档

- `AGENTS.md`：人类和 AI agent 的项目规则
- `CONTRIBUTING.md`：贡献流程和 PR 要求
- `LICENSE`：默认 MIT 许可证，派生项目可按需要替换
- `SECURITY.md`：安全漏洞报告流程
- `docs/conventions.md`：语言、文档、测试、安全等项目约定
- `docs/git.md`：Git 工作流和 commit 规范
- `docs/SAD.md`：可选，软件架构文档（Software Architecture Document）
- `docs/SDD.md`：可选，软件设计文档（Software Design Document）
- `docs/adr/`：架构决策
- `docs/rfcs/`：提案
- `docs/specs/`：设计文档
- `docs/plans/`：实施计划
- `docs/templates/`：局部规则、硬件设计、供应商记录、SOP 模板
- `docs/hardware/`：可选，硬件设计和验证记录
- `docs/production/`：可选，生产、发布、验收、回滚流程
- `docs/suppliers/`：可选，供应商和第三方交付物记录
- `docs/sop/`：可选，标准作业流程

## Commit 规范

默认采用 Conventional Commits 风格，subject 中文优先，type/scope 保持英文，例如 `feat(api): 支持 OAuth callback`。详细规则见 `docs/git.md`。
