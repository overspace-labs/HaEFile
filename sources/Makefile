# ============================================================
#  HaE VSCode 扩展 Makefile
#  一键安装依赖 / 编译 / 打包 .vsix / 安装到本地 VSCode
# ============================================================

SHELL := /bin/sh

# 项目根目录（Makefile 所在目录）
ROOT := $(abspath $(dir $(lastword $(MAKEFILE_LIST))))

# 工具命令
NPM  := npm
NPX  := npx
VSCE := $(NPX) vsce

# 从 package.json 动态读取扩展名与版本号，构造 .vsix 路径
NAME    := $(shell node -p "require('./package.json').name")
VERSION := $(shell node -p "require('./package.json').version")
BUILD_DIR := build
VSIX      := $(BUILD_DIR)/$(NAME)-$(VERSION).vsix

# 本地 VSCode CLI：优先 code，其次 code-insiders
CODE := $(shell command -v code 2>/dev/null || command -v code-insiders 2>/dev/null)

# 关键目录
WEBVIEW_DIR   := webview-ui
WEBVIEW_DIST  := $(WEBVIEW_DIR)/dist
OUT_DIR       := out

# 终端颜色（仅交互式终端输出）
ifeq ($(shell [ -t 1 ] && echo 1),1)
C_RESET := \033[0m
C_BOLD  := \033[1m
C_GREEN := \033[32m
C_YELLOW:= \033[33m
C_CYAN  := \033[36m
C_RED   := \033[31m
else
C_RESET :=
C_BOLD  :=
C_GREEN :=
C_YELLOW:=
C_CYAN  :=
C_RED   :=
endif

# 默认目标：打印帮助
.DEFAULT_GOAL := package

# ---------------- 依赖安装 ----------------
.PHONY: install-deps install-webview install-all
install-deps: ## 安装根目录依赖
	@printf "$(C_CYAN)==> 安装根目录依赖$(C_RESET)\n"
	$(NPM) install

install-webview: ## 安装 webview-ui 依赖
	@printf "$(C_CYAN)==> 安装 webview-ui 依赖$(C_RESET)\n"
	cd $(WEBVIEW_DIR) && $(NPM) install

install-all: install-deps install-webview ## 安装全部依赖（根 + webview）
	@printf "$(C_GREEN)==> 依赖安装完成$(C_RESET)\n"

# ---------------- 构建 ----------------
.PHONY: build build-webview compile watch watch-webview
build: ## 完整构建（清理 + 构建 webview + 编译 TS）
	@printf "$(C_CYAN)==> 完整构建$(C_RESET)\n"
	$(NPM) run build

build-webview: ## 仅构建 webview-ui（vite build）
	$(NPM) run build:webview

compile: ## 仅编译 TypeScript（tsc）
	$(NPM) run compile

watch: ## 监听 TypeScript 变更
	$(NPM) run watch

watch-webview: ## 监听 webview-ui 变更（vite dev）
	$(NPM) run watch:webview

# ---------------- 代码质量 ----------------
.PHONY: lint lint-fix format format-check test
lint: ## 运行 ESLint 检查
	$(NPM) run lint

lint-fix: ## ESLint 自动修复
	$(NPM) run lint:fix

format: ## Prettier 格式化
	$(NPM) run format

format-check: ## Prettier 检查（不修改）
	$(NPM) run format:check

test: ## 运行扩展测试（vscode-test）
	$(NPM) run test

# ---------------- 打包 / 安装 / 发布 ----------------
.PHONY: package package-pre install publish
package: ## 构建并打包到 build/*.vsix（自动触发 prepublish 构建）
	@printf "$(C_CYAN)==> 打包 VSIX → $(BUILD_DIR)/$(C_RESET)\n"
	@mkdir -p $(BUILD_DIR)
	$(VSCE) package --no-yarn -o $(VSIX)
	@printf "$(C_GREEN)==> 产出 $(VSIX)$(C_RESET)\n"
	@ls -lh $(VSIX)

package-pre: ## 打包 pre-release 版本到 build/
	@printf "$(C_CYAN)==> 打包 VSIX (pre-release) → $(BUILD_DIR)/$(C_RESET)\n"
	@mkdir -p $(BUILD_DIR)
	$(VSCE) package --no-yarn --pre-release -o $(BUILD_DIR)/$(NAME)-$(VERSION)-pre.vsix

install: package ## 构建并打包，然后安装到本地 VSCode（code --install-extension）
	@printf "$(C_CYAN)==> 安装到本地 VSCode$(C_RESET)\n"
ifeq ($(strip $(CODE)),)
	$(error 未找到 code 或 code-insiders 命令，请确认 VSCode CLI 已加入 PATH)
endif
	$(CODE) --install-extension $(VSIX) --force
	@printf "$(C_GREEN)==> 已安装到本地 VSCode：$(VERSION)$(C_RESET)\n"

publish: build ## 发布到 VS Code Marketplace（需已登录）
	@printf "$(C_CYAN)==> 发布到 Marketplace$(C_RESET)\n"
	$(VSCE) publish --no-yarn

# ---------------- 版本号 ----------------
.PHONY: patch minor major
patch: ## 版本号 +0.0.1 后打包到 build/
	$(NPM) version patch --no-git-tag-version
	$(MAKE) package

minor: ## 版本号 +0.1.0 后打包到 build/
	$(NPM) version minor --no-git-tag-version
	$(MAKE) package

major: ## 版本号 +1.0.0 后打包到 build/
	$(NPM) version major --no-git-tag-version
	$(MAKE) package

# ---------------- 清理 ----------------
.PHONY: clean clean-all
clean: ## 清理构建产物（out + webview dist + build）
	@printf "$(C_YELLOW)==> 清理构建产物$(C_RESET)\n"
	rm -rf $(OUT_DIR) $(WEBVIEW_DIST) $(BUILD_DIR)

clean-all: clean ## 清理构建产物 + 依赖目录
	rm -rf node_modules $(WEBVIEW_DIR)/node_modules

# ---------------- 一键流程 ----------------
.PHONY: dist
dist: install-all package ## 一键：安装依赖 → 构建并打包到 build/

# ---------------- 帮助 ----------------
.PHONY: help
help: ## 显示本帮助
	@printf "$(C_BOLD)HaE VSCode 扩展 Makefile$(C_RESET)\n\n"
	@printf "$(C_BOLD)常用目标：$(C_RESET)\n"
	@awk 'BEGIN {FS = ":.*## "} /^[a-zA-Z_-]+:.*## / \
		{printf "  $(C_GREEN)%-15s$(C_RESET) %s\n", $$1, $$2}' $(MAKEFILE_LIST)
	@printf "\n$(C_BOLD)示例：$(C_RESET)\n"
	@printf "  make install-all   # 安装全部依赖\n"
	@printf "  make build         # 编译整个插件\n"
	@printf "  make package       # 打包到 build/*.vsix\n"
	@printf "  make install       # 一键：构建 + 打包 + 安装到本地 VSCode\n"
	@printf "  make dist          # 一键：安装依赖 + 打包到 build/\n"
	@printf "  make patch         # 升补丁号并打包到 build/\n"
