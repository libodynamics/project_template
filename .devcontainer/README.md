<!-- Copyright The Project Template Contributors -->

# Dev Container

本目录负责模板项目的开发容器和公司通用基础镜像。

模板仓库发布的基础镜像：

```Dockerfile
FROM ghcr.io/libodynamics/project_template/devcontainer:latest
```

公司策略是统一使用 `latest`。派生项目默认继承这个镜像，并在自己的 `.devcontainer/Dockerfile` 中追加项目专用工具链、SDK、模拟器、数据库客户端、硬件工具或缓存配置。

如果该基础镜像保持为 private GHCR package，派生项目 CI 不能依赖匿名 pull。先在基础镜像 package 的 `Manage Actions access` 中给派生仓库授予 `Read` 权限，然后在需要构建 `.devcontainer/Dockerfile` 或直接使用该镜像的 workflow job 中保留：

```yaml
permissions:
  contents: read
  packages: read

steps:
  - uses: actions/checkout@v6
  - uses: docker/login-action@v4
    with:
      registry: ghcr.io
      username: ${{ github.actor }}
      password: ${{ secrets.GITHUB_TOKEN }}
```

非 GitHub CI 或开发者本机需要拉取 private 镜像时，使用带 `read:packages` scope 的个人或机器人 token 登录 `ghcr.io`。真实 token 只能进入 CI secret、本机凭据管理或一次性 shell 环境变量，不能写入 Dockerfile、devcontainer 配置或仓库文档示例。

模板仓库固定使用以下 Docker 名称：

| 用途 | 名称 |
|------|------|
| 基础 Dev Container 镜像 | `ghcr.io/libodynamics/project_template/devcontainer:latest` |
| 本地派生 Dev Container 镜像 | `project-template-devcontainer:latest` |
| Dev Container 显示名 | `project-template-devcontainer` |
| 常驻 Dev Container 容器名 | `project-template-devcontainer-{username}-{branch}` |
| 项目服务/应用运行时容器名 | `project-template-{service}-{username}-{branch}` |

派生项目初始化时必须把这些名称替换为项目自己的稳定名称，并保持 `devcontainer.json`、README、CI 和手动 `docker run` 示例一致。容器名最后一段必须使用当前具名 Git 分支，分支名中的 `/`、空格和其他特殊字符必须替换为 `-`；detached HEAD 状态不得启动常驻 Dev Container 或项目运行时容器，必须先切换到具名分支。

## 文件分工

- `devcontainer.json`
- `Dockerfile`：派生项目默认入口，基于 `ghcr.io/libodynamics/project_template/devcontainer:latest`
- `base.Dockerfile`：仅供模板仓库维护通用基础镜像，由 CI 发布到 GHCR；派生项目默认删除
- `compose.yaml`：需要本地数据库、缓存、消息队列等服务时再添加
- `scripts/`：容器初始化脚本

除非项目明确发布生产容器镜像，否则不要在仓库根目录添加 Dockerfile。

派生项目只有在需要维护自己的组织级或项目级基础镜像时才保留 `base.Dockerfile`。保留时必须同步更新 README、CI 和镜像命名表，说明镜像发布到哪里、谁维护、何时重建、如何回滚；否则应删除 `base.Dockerfile` 和基础镜像发布 workflow，避免贡献者误以为每个项目都要构建基础镜像。

## 基础镜像工具

模板默认安装跨项目常用工具：

- 基础构建：`build-essential`、`pkg-config`、`cmake`、`ninja-build`、`make`
- C/C++/FFI：`clang`、`llvm`、`lld`、`libclang-dev`、`libssl-dev`
- Rust：`rustup stable`、`clippy`、`rustfmt`、`llvm-tools-preview`
- Shell 与排障：`zsh`、`vim`、`less`、`tree`、`htop`
- 文本与 JSON：`jq`、`ripgrep`、`fd`、`bat`
- 脚本与文档：`python3`、`pre-commit`、Ubuntu 26.04 archive 提供的 `nodejs`、`npm@latest`、`graphviz`、`@mermaid-js/mermaid-cli`、latest upstream `plantuml`
- Dev Container 辅助：`@devcontainers/cli`
- 依赖维护：`npm-check-updates`
- 协作：`git`、`git-lfs`、`openssh-client`、`gh`
- Locale：`en_US.UTF-8`

默认交互用户是 `dev`，默认 shell 是 `zsh`，并允许免密码 `sudo`。Dockerfile 构建阶段仍使用 root；派生项目需要安装系统包时，继续在 `.devcontainer/Dockerfile` 中使用 `RUN apt-get ...`。

## 派生项目可选工具

不要把只有单个项目使用的工具强制装入基础镜像。派生项目按需要在 `.devcontainer/Dockerfile` 中追加：

```Dockerfile
FROM ghcr.io/libodynamics/project_template/devcontainer:latest

# 嵌入式 / 内核 / 固件开发常见工具
RUN apt-get update && apt-get install --no-install-recommends -y \
    qemu-system-arm \
    qemu-system-misc \
    gdb-multiarch \
    device-tree-compiler \
    u-boot-tools \
    dosfstools \
    gcc-riscv64-linux-gnu \
    binutils-riscv64-linux-gnu \
    gcc-aarch64-linux-gnu \
    binutils-aarch64-linux-gnu \
    gcc-arm-linux-gnueabihf \
    binutils-arm-linux-gnueabihf \
    && rm -rf /var/lib/apt/lists/*
```

项目专用工具也应由派生项目按需添加，例如 Tauri Linux 依赖、交叉编译器、RT 工具、CAN 工具、MATLAB/Simulink 外部脚本入口、数据库客户端或硬件 SDK。

## 手动 Docker 验证

维护模板基础镜像时，宿主机只负责镜像构建、容器创建和容器启动。默认先启动一个可复用的常驻 Dev Container；后续所有检查通过 `docker exec` 在该容器中运行。

```bash
docker build --pull -f .devcontainer/base.Dockerfile -t ghcr.io/libodynamics/project_template/devcontainer:latest .devcontainer
docker build --pull=false -f .devcontainer/Dockerfile -t project-template-devcontainer:latest .devcontainer
DEVCONTAINER_USER="$(id -un | sed -E 's/[^[:alnum:]_.-]+/-/g; s/^-+//; s/-+$//')"
DEVCONTAINER_BRANCH="$(git branch --show-current | sed -E 's/[^[:alnum:]_.-]+/-/g; s/^-+//; s/-+$//')"
if [ -z "$DEVCONTAINER_BRANCH" ]; then echo "detached HEAD is not allowed for the devcontainer name" >&2; exit 1; fi
export DEVCONTAINER_NAME="project-template-devcontainer-${DEVCONTAINER_USER}-${DEVCONTAINER_BRANCH}"
docker inspect "$DEVCONTAINER_NAME" >/dev/null 2>&1 || docker run -d --name "$DEVCONTAINER_NAME" \
  --mount "type=bind,src=$(pwd),dst=/workspace" \
  -w /workspace \
  project-template-devcontainer:latest sleep infinity
if [ "$(docker inspect -f '{{.State.Running}}' "$DEVCONTAINER_NAME")" != "true" ]; then docker start "$DEVCONTAINER_NAME" >/dev/null; fi
docker exec "$DEVCONTAINER_NAME" bash -lc '
    git config --global --add safe.directory /workspace &&
    git status --short --branch &&
    rg -n "TODO|YYYY-MM-DD|@TODO-owner" . &&
    pre-commit run --all-files &&
    rustc --version &&
    node --version &&
    npm --version &&
    devcontainer --version &&
    mmdc --version &&
    plantuml -version &&
    ncu --version
  '
```

容器已经存在但停止时，先运行 `docker start "$DEVCONTAINER_NAME"`，再运行 `docker exec ...`；新 shell 中运行前必须先按上方规则重新设置同一个 `DEVCONTAINER_NAME`。镜像、挂载或工作区路径变化时，确认旧容器没有需要保留的状态，删除旧容器并重新按本节创建。

手动 `docker run`、Compose volume 和 Dev Container mount 只能把当前仓库或仓库内子目录挂载到容器项目工作区内，例如 `$PWD:/workspace`。`docker run` 只用于创建常驻后台容器，不得直接执行项目命令；项目检查、构建、测试、生成、打包和发布命令必须通过 `docker exec "$DEVCONTAINER_NAME" ...` 进入已启动容器执行。不要把用户主目录、上级目录、系统目录或无关临时目录挂入容器。宿主机调用 Docker 编排后，需要保留的产物应写到项目目录下的 `build/`、`dist/`、`target/`、`out/` 或项目声明的产物目录，确保退出容器后仍能在宿主机项目目录中看到。

## 产物路径

Docker/Dev Container 中生成的编译、打包、部署或文档产物，必须写入当前项目 bind mount 内的声明目录。README 中应同时列出宿主机路径和容器内路径，例如宿主机 `dist/` 对应容器 `/workspace/dist`。发布、部署、验收和回滚命令必须读取这个宿主机可见目录中的产物，不要依赖容器临时文件系统、匿名 volume、`/tmp`、`/home/dev`、上级目录或项目外缓存中的唯一副本。

CI 会在 `main` 分支和每周定时任务中把基础镜像发布为：

```text
ghcr.io/libodynamics/project_template/devcontainer:latest
```

## 验证环境

以下命令只在已启动的常驻 Dev Container 内执行；宿主机侧使用 `docker exec "$DEVCONTAINER_NAME" ...` 进入容器。

```bash
cat /etc/os-release
git --version
gh --version
python3 --version
node --version
npm --version
rustc --version
cargo --version
cargo clippy --version
rustfmt --version
pre-commit --version
rg --version
fd --version
bat --version
devcontainer --version
mmdc --version
plantuml -version
ncu --version
```
