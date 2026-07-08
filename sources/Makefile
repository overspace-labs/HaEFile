SHELL := /bin/sh

# 工具命令
NPM  := npm
NPX  := npx
VSCE := $(NPX) --no-install vsce

# 从 package.json 动态读取扩展名与版本号，构造 .vsix 路径
NAME    := $(shell node -p "require('./package.json').name")
VERSION := $(shell node -p "require('./package.json').version")
BUILD_DIR := build
VSIX      := $(BUILD_DIR)/$(NAME)-$(VERSION).vsix

# 本地 VSCode CLI：默认优先 code，其次 code-insiders
CODE := $(shell command -v code 2>/dev/null)
CODE_INSIDERS := $(shell command -v code-insiders 2>/dev/null)
EDITOR_CLI ?= $(or $(CODE),$(CODE_INSIDERS))

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
.DEFAULT_GOAL := help

# ---------------- 依赖安装 ----------------
.PHONY: install-deps install-webview install-all
install-deps: ## Install root dependencies
	@printf "$(C_CYAN)==> Installing root dependencies$(C_RESET)\n"
	$(NPM) ci

install-webview: ## Install webview-ui dependencies
	@printf "$(C_CYAN)==> Installing webview-ui dependencies$(C_RESET)\n"
	cd $(WEBVIEW_DIR) && $(NPM) ci

install-all: install-deps install-webview ## Install all dependencies (root + webview)
	@printf "$(C_GREEN)==> Dependencies installed$(C_RESET)\n"

# ---------------- 构建 ----------------
.PHONY: build build-webview compile watch watch-webview
build: ## Build the extension (clean + webview + TypeScript)
	@printf "$(C_CYAN)==> Building extension$(C_RESET)\n"
	$(NPM) run build

build-webview: ## Build webview-ui only
	$(NPM) run build:webview

compile: ## Compile TypeScript only
	$(NPM) run compile

watch: ## Watch TypeScript changes
	$(NPM) run watch

watch-webview: ## Watch webview-ui changes
	$(NPM) run watch:webview

# ---------------- 代码质量 ----------------
.PHONY: lint lint-fix format format-check test
lint: ## Run ESLint
	$(NPM) run lint

lint-fix: ## Run ESLint with auto-fix
	$(NPM) run lint:fix

format: ## Format with Prettier
	$(NPM) run format

format-check: ## Check formatting with Prettier
	$(NPM) run format:check

test: ## Run extension tests
	$(NPM) run test

# ---------------- 打包 / 安装 / 发布 ----------------
.PHONY: package package-pre install publish
package: ## Build and package to build/*.vsix
	@printf "$(C_CYAN)==> Packaging VSIX -> $(BUILD_DIR)/$(C_RESET)\n"
	@mkdir -p $(BUILD_DIR)
	$(VSCE) package --no-yarn -o $(VSIX)
	@printf "$(C_GREEN)==> Created $(VSIX)$(C_RESET)\n"
	@ls -lh $(VSIX)

package-pre: ## Package pre-release VSIX to build/
	@printf "$(C_CYAN)==> Packaging pre-release VSIX -> $(BUILD_DIR)$(C_RESET)\n"
	@mkdir -p $(BUILD_DIR)
	$(VSCE) package --no-yarn --pre-release -o $(BUILD_DIR)/$(NAME)-$(VERSION)-pre.vsix

install: install-all package ## Install to the first available VS Code CLI
	@printf "$(C_CYAN)==> Installing to: $(EDITOR_CLI)$(C_RESET)\n"
ifeq ($(strip $(EDITOR_CLI)),)
	$(error No VS Code CLI found. Install 'code' or 'code-insiders', then retry after configuring PATH)
endif
	$(EDITOR_CLI) --install-extension $(VSIX) --force
	@printf "$(C_GREEN)==> Installed extension version $(VERSION)$(C_RESET)\n"

publish: build ## Publish to the VS Code Marketplace
	@printf "$(C_CYAN)==> Publishing to Marketplace$(C_RESET)\n"
	$(VSCE) publish --no-yarn

# ---------------- 版本号 ----------------
.PHONY: patch minor major
patch: ## Bump patch version and package
	$(NPM) version patch --no-git-tag-version
	$(MAKE) package

minor: ## Bump minor version and package
	$(NPM) version minor --no-git-tag-version
	$(MAKE) package

major: ## Bump major version and package
	$(NPM) version major --no-git-tag-version
	$(MAKE) package

# ---------------- 清理 ----------------
.PHONY: clean clean-all
clean: ## Remove build artifacts
	@printf "$(C_YELLOW)==> Cleaning build artifacts$(C_RESET)\n"
	rm -rf $(OUT_DIR) $(WEBVIEW_DIST) $(BUILD_DIR)

clean-all: clean ## Remove build artifacts and dependencies
	rm -rf node_modules $(WEBVIEW_DIR)/node_modules

# ---------------- 一键流程 ----------------
.PHONY: dist
dist: install-all package ## Install dependencies and package to build/

# ---------------- 帮助 ----------------
.PHONY: help
help: ## Show help
	@printf "$(C_BOLD)HaE VSCode Extension Makefile$(C_RESET)\n\n"
	@printf "$(C_BOLD)Common targets:$(C_RESET)\n"
	@awk 'BEGIN {FS = ":.*## "} /^[a-zA-Z_-]+:.*## / \
		{printf "  $(C_GREEN)%-15s$(C_RESET) %s\n", $$1, $$2}' $(MAKEFILE_LIST)
	@printf "\n$(C_BOLD)Examples:$(C_RESET)\n"
	@printf "  make build             # Build the extension\n"
	@printf "  make package           # Create build/*.vsix\n"
	@printf "  make install           # Auto-detect VS Code / Insiders and install\n"
	@printf "  make dist              # Install deps and package\n"
