SHELL := /bin/sh

# --- Variables ---

NPM  := npm
VSCE := npx --no-install vsce

NAME    := $(shell node -p "require('./package.json').name")
VERSION := $(shell node -p "require('./package.json').version")
BUILD_DIR := build
VSIX      := $(BUILD_DIR)/$(NAME)-$(VERSION).vsix

CODE        := $(shell command -v code 2>/dev/null)
CODE_INSIDERS := $(shell command -v code-insiders 2>/dev/null)
EDITOR_CLI  ?= $(or $(CODE),$(CODE_INSIDERS))

WEBVIEW_DIR  := webview-ui
WEBVIEW_DIST := $(WEBVIEW_DIR)/dist

.PHONY: help install-all build package install clean clean-all \
        watch lint test publish dist patch minor major

# --- Help ---

help:
	@echo "Usage:"
	@echo "  make build       Build the extension"
	@echo "  make package     Package VSIX to build/"
	@echo "  make install     Install deps, package, and install to VS Code"
	@echo "  make clean       Remove build artifacts"
	@echo "  make dist        Install deps and package"

# --- Install ---

install-all:
	$(NPM) ci
	cd $(WEBVIEW_DIR) && $(NPM) ci

# --- Build ---

build:
	$(NPM) run build

watch:
	$(NPM) run watch

lint:
	$(NPM) run lint

test:
	$(NPM) run test

# --- Package ---

package:
	@mkdir -p $(BUILD_DIR)
	$(VSCE) package --no-yarn -o $(VSIX)

install: install-all package
ifeq ($(strip $(EDITOR_CLI)),)
	$(error No VS Code CLI found. Install 'code' or 'code-insiders' and configure PATH)
endif
	$(EDITOR_CLI) --install-extension $(VSIX) --force

publish: build
	$(VSCE) publish --no-yarn

dist: install-all package

# --- Version ---

patch:
	$(NPM) version patch --no-git-tag-version
	$(MAKE) package

minor:
	$(NPM) version minor --no-git-tag-version
	$(MAKE) package

major:
	$(NPM) version major --no-git-tag-version
	$(MAKE) package

# --- Clean ---

clean:
	rm -rf out $(WEBVIEW_DIST) $(BUILD_DIR)

clean-all: clean
	rm -rf node_modules $(WEBVIEW_DIR)/node_modules
