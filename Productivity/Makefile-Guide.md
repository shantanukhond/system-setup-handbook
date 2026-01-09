<div align="center">

# 🔧 Makefile Guide

**Automate Your Development Workflow with Makefiles**

[![Make](https://img.shields.io/badge/Make-Automation-blue)]()
[![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-green)]()

*Learn how to create and use Makefiles to streamline your development tasks and operations*

---

</div>

## 📋 Overview

This guide will teach you how to:

- ✅ Create and use Makefiles for task automation
- ✅ Define custom commands and targets
- ✅ Manage variables and parameters
- ✅ Organize complex workflows
- ✅ Share standardized commands across teams

> 💡 **Note:** Makefiles are a powerful tool for automating repetitive tasks, running complex commands, and standardizing development workflows. They work on Unix-like systems and can save hours of typing and reduce errors.

---

## 🚀 Getting Started

### Step 1: Understanding Makefiles

A Makefile is a special file containing a set of directives used by the `make` build automation tool. It consists of **targets**, **dependencies**, and **commands**.

**Basic Syntax:**
```makefile
target: dependencies
	command
	command
```

**Important:** Commands must be indented with a **TAB** character, not spaces!

### Step 2: Create Your First Makefile

Create a file named `Makefile` (capital M, no extension) in your project root:

```bash
touch Makefile
vi Makefile
```

**Example Basic Makefile:**
```makefile
hello:
	echo "Hello, World!"
```

Run it:
```bash
make hello
```

---

## 🎯 Common Use Cases

### Step 3: Project Management Commands

**Example Makefile for a Node.js project:**
```makefile
.PHONY: install build test clean run dev help

# Default target (runs when you just type 'make')
.DEFAULT_GOAL := help

# Variables
NODE_ENV ?= development
PORT ?= 3000
APP_NAME ?= myapp

help: ## Show this help message
	@echo 'Usage: make [target]'
	@echo ''
	@echo 'Available targets:'
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}'

install: ## Install dependencies
	npm install
	@echo "Dependencies installed"

build: ## Build the project
	NODE_ENV=production npm run build
	@echo "Build complete"

test: ## Run tests
	npm test
	@echo "Tests completed"

clean: ## Clean build artifacts
	rm -rf dist/
	rm -rf node_modules/.cache
	@echo "Cleanup complete"

run: ## Run the application
	npm start

dev: ## Run development server
	npm run dev

docker-build: ## Build Docker image
	docker build -t $(APP_NAME) .
	@echo "Docker image built: $(APP_NAME)"

docker-run: ## Run Docker container
	docker run -p $(PORT):$(PORT) $(APP_NAME)
```

---

### Step 4: Server Management Commands

**Example Makefile for server operations:**
```makefile
.PHONY: deploy restart status logs backup clean

SERVER_USER ?= ubuntu
SERVER_HOST ?= example.com
SERVER_PORT ?= 7022
SSH_KEY ?= ~/.ssh/server_key

SSH = ssh -p $(SERVER_PORT) -i $(SSH_KEY) $(SERVER_USER)@$(SERVER_HOST)

deploy: ## Deploy to server
	@echo "Deploying to $(SERVER_HOST)..."
	rsync -avz -e "ssh -p $(SERVER_PORT) -i $(SSH_KEY)" \
		--exclude 'node_modules' \
		--exclude '.git' \
		./ $(SERVER_USER)@$(SERVER_HOST):/var/www/app/
	$(SSH) 'cd /var/www/app && npm install --production'
	$(SSH) 'pm2 restart app || pm2 start app.js --name app'
	@echo "Deployment complete"

restart: ## Restart server services
	$(SSH) 'sudo systemctl restart nginx'
	$(SSH) 'sudo systemctl restart app'
	@echo "Services restarted"

status: ## Check server status
	$(SSH) 'sudo systemctl status nginx'
	$(SSH) 'sudo systemctl status app'

logs: ## View application logs
	$(SSH) 'tail -f /var/log/app/error.log'

backup: ## Backup server data
	@echo "Creating backup..."
	$(SSH) 'tar -czf /tmp/backup-$(shell date +%Y%m%d).tar.gz /var/www/app/data'
	scp -P $(SERVER_PORT) -i $(SSH_KEY) \
		$(SERVER_USER)@$(SERVER_HOST):/tmp/backup-*.tar.gz ./backups/
	@echo "Backup complete"

clean: ## Clean old backups (older than 30 days)
	find ./backups -name "*.tar.gz" -mtime +30 -delete
	@echo "Old backups cleaned"
```

---

### Step 5: Database Management

**Example Makefile for database operations:**
```makefile
.PHONY: db-migrate db-rollback db-seed db-reset db-backup db-restore

DB_NAME ?= myapp
DB_USER ?= dbuser
DB_HOST ?= localhost
DB_PORT ?= 5432

db-migrate: ## Run database migrations
	@echo "Running migrations..."
	knex migrate:latest
	@echo "Migrations complete"

db-rollback: ## Rollback last migration
	@echo "Rolling back migration..."
	knex migrate:rollback
	@echo "Rollback complete"

db-seed: ## Seed database
	@echo "Seeding database..."
	knex seed:run
	@echo "Seeding complete"

db-reset: ## Reset database (migrate + seed)
	@echo "Resetting database..."
	knex migrate:rollback --all
	knex migrate:latest
	knex seed:run
	@echo "Database reset complete"

db-backup: ## Backup database
	@echo "Creating database backup..."
	pg_dump -h $(DB_HOST) -p $(DB_PORT) -U $(DB_USER) $(DB_NAME) > \
		backups/db-backup-$(shell date +%Y%m%d-%H%M%S).sql
	@echo "Database backup complete"

db-restore: ## Restore database from backup
	@echo "Available backups:"
	@ls -lh backups/*.sql
	@read -p "Enter backup file name: " file; \
	psql -h $(DB_HOST) -p $(DB_PORT) -U $(DB_USER) $(DB_NAME) < backups/$$file
	@echo "Database restored"
```

---

### Step 6: Docker Operations

**Example Makefile for Docker workflows:**
```makefile
.PHONY: docker-build docker-run docker-stop docker-rm docker-logs docker-ps

IMAGE_NAME ?= myapp
CONTAINER_NAME ?= myapp-container
PORT ?= 3000

docker-build: ## Build Docker image
	@echo "Building Docker image: $(IMAGE_NAME)"
	docker build -t $(IMAGE_NAME):latest .
	@echo "Build complete"

docker-run: ## Run Docker container
	@echo "Starting container: $(CONTAINER_NAME)"
	docker run -d \
		--name $(CONTAINER_NAME) \
		-p $(PORT):$(PORT) \
		--env-file .env \
		$(IMAGE_NAME):latest
	@echo "Container started"

docker-stop: ## Stop Docker container
	@echo "Stopping container: $(CONTAINER_NAME)"
	docker stop $(CONTAINER_NAME)
	@echo "Container stopped"

docker-rm: docker-stop ## Remove Docker container
	@echo "Removing container: $(CONTAINER_NAME)"
	docker rm $(CONTAINER_NAME)
	@echo "Container removed"

docker-logs: ## View container logs
	docker logs -f $(CONTAINER_NAME)

docker-ps: ## List running containers
	docker ps

docker-clean: ## Clean up Docker resources
	@echo "Cleaning up Docker resources..."
	docker system prune -f
	docker volume prune -f
	@echo "Cleanup complete"
```

---

### Step 7: Advanced Features

#### 7.1 Conditional Commands

```makefile
.PHONY: deploy

ENV ?= staging

deploy:
ifeq ($(ENV),production)
	@echo "Deploying to PRODUCTION..."
	@read -p "Are you sure? [y/N] " -n 1 -r; \
	echo; \
	if [[ $$REPLY =~ ^[Yy]$$ ]]; then \
		echo "Deploying..."; \
	fi
else
	@echo "Deploying to $(ENV)..."
	# Deploy commands
endif
```

Usage:
```bash
make deploy ENV=production
make deploy  # defaults to staging
```

#### 7.2 Multiple Targets

```makefile
.PHONY: setup install configure

setup: install configure ## Complete setup process
	@echo "Setup complete"

install: ## Install dependencies
	npm install

configure: ## Configure environment
	cp .env.example .env
	@echo "Configuration complete"
```

#### 7.3 Including Other Makefiles

```makefile
include docker.mk
include deploy.mk
include test.mk
```

#### 7.4 Functions and Loops

```makefile
SERVERS = server1.com server2.com server3.com

deploy-all: ## Deploy to all servers
	@for server in $(SERVERS); do \
		echo "Deploying to $$server..."; \
		rsync -avz ./ user@$$server:/var/www/app/; \
	done
	@echo "Deployment to all servers complete"
```

---

## 📝 Complete Example Makefile

Here's a comprehensive example combining multiple concepts:

```makefile
.PHONY: help install build test deploy clean docker-build docker-run

.DEFAULT_GOAL := help

# Variables
APP_NAME ?= myapp
VERSION ?= 1.0.0
ENV ?= development
PORT ?= 3000
SERVER_HOST ?= example.com
SERVER_USER ?= ubuntu

# Colors for output
RED := \033[0;31m
GREEN := \033[0;32m
YELLOW := \033[0;33m
NC := \033[0m # No Color

help: ## Show this help message
	@echo "$(GREEN)Available commands:$(NC)"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | \
		awk 'BEGIN {FS = ":.*?## "}; {printf "  $(YELLOW)%-20s$(NC) %s\n", $$1, $$2}'

install: ## Install dependencies
	@echo "$(GREEN)Installing dependencies...$(NC)"
	npm install
	@echo "$(GREEN)✓ Dependencies installed$(NC)"

build: ## Build the application
	@echo "$(GREEN)Building application...$(NC)"
	npm run build
	@echo "$(GREEN)✓ Build complete$(NC)"

test: ## Run tests
	@echo "$(GREEN)Running tests...$(NC)"
	npm test
	@echo "$(GREEN)✓ Tests complete$(NC)"

lint: ## Run linter
	@echo "$(GREEN)Running linter...$(NC)"
	npm run lint
	@echo "$(GREEN)✓ Linting complete$(NC)"

deploy: build ## Deploy to server
	@echo "$(GREEN)Deploying to $(SERVER_HOST)...$(NC)"
	rsync -avz --exclude 'node_modules' --exclude '.git' \
		./ $(SERVER_USER)@$(SERVER_HOST):/var/www/$(APP_NAME)/
	ssh $(SERVER_USER)@$(SERVER_HOST) \
		'cd /var/www/$(APP_NAME) && npm install --production && pm2 restart $(APP_NAME)'
	@echo "$(GREEN)✓ Deployment complete$(NC)"

clean: ## Clean build artifacts
	@echo "$(YELLOW)Cleaning up...$(NC)"
	rm -rf dist/ build/ node_modules/.cache
	@echo "$(GREEN)✓ Cleanup complete$(NC)"

docker-build: ## Build Docker image
	@echo "$(GREEN)Building Docker image...$(NC)"
	docker build -t $(APP_NAME):$(VERSION) .
	docker tag $(APP_NAME):$(VERSION) $(APP_NAME):latest
	@echo "$(GREEN)✓ Docker image built$(NC)"

docker-run: ## Run Docker container
	@echo "$(GREEN)Starting Docker container...$(NC)"
	docker run -d --name $(APP_NAME) -p $(PORT):$(PORT) $(APP_NAME):latest
	@echo "$(GREEN)✓ Container started$(NC)"

setup: install build ## Complete project setup
	@echo "$(GREEN)✓ Project setup complete$(NC)"
```

---

## ✅ Best Practices

1. ✅ **Use `.PHONY`** - Declare non-file targets to avoid conflicts
2. ✅ **Set `.DEFAULT_GOAL`** - Define what runs with just `make`
3. ✅ **Use variables** - Make your Makefile reusable and configurable
4. ✅ **Add help target** - Document your targets with descriptions
5. ✅ **Use `@` prefix** - Hide command echo for cleaner output
6. ✅ **Organize targets** - Group related targets together
7. ✅ **Use meaningful names** - Make target names self-documenting
8. ✅ **Test your Makefile** - Verify all targets work correctly

---

## 🔧 Troubleshooting

### "Missing separator" Error

**Problem:** Commands must be indented with TAB, not spaces.

**Solution:**
```bash
# In vim, set tab characters
:set noexpandtab
:set tabstop=4

# Or use this to check for tabs
cat -A Makefile | grep -E '^[^ ]* '
```

### Variables Not Expanding

**Use `$(VARIABLE)` syntax:**
```makefile
VAR = value
target:
	echo $(VAR)  # Correct
	echo $$VAR   # Incorrect (shell variable)
```

### Target Always Runs

**Use `.PHONY` for non-file targets:**
```makefile
.PHONY: clean deploy
```

---

## 📚 Additional Resources

- [GNU Make Manual](https://www.gnu.org/software/make/manual/)
- [Makefile Tutorial](https://makefiletutorial.com/)
- [Awesome Makefiles](https://github.com/mbcrawfo/GenericMakefile)

---

<div align="center">

**🔧 Your Makefile workflow is now ready!**

[← Back to Handbook](../README.md)

</div>

