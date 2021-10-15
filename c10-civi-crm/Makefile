SHELL:=/bin/bash
.ONESHELL:
.SHELLFLAGS := -eu -o pipefail -c
.DELETE_ON_ERROR:
MAKEFLAGS += --warn-undefined-variables
MAKEFLAGS += --no-builtin-rules
.PHONY: help
.DEFAULT_GOAL := help

ifndef NO_COLOR
YELLOW=\033[0;33m
CYAN=\033[1;36m
# no color
NC=\033[0m
endif

include .env
$(eval export $(grep '=' .env | cut -d= -f1))

help:
	@echo -e "$$(grep -hE '^\S+:.*##' $(MAKEFILE_LIST) | sed -e 's/:.*##\s*/:/' -e 's/^\(.\+\):\(.*\)/\\x1b[36m\1\\x1b[m:\2/' | column -c2 -t -s :)"

setup: ## Setup docker volume dirs
	mkdir src 2>/dev/null || true
	mkdir -p data/docker-containers-volumes/var/lib/mysql
	mkdir -p data/docker-containers-volumes/var/www/html/web/sites/default/files
	sudo chown -R :33 data/docker-containers-volumes/var/www/html/web/

prune: ## Drop persistent data dirs
	sudo rm -rfI data/ src/

up:    ## Launch local stack
	@echo -e "\n$(YELLOW) Launch docker-compose stack \n"
	docker-compose up -d
	@echo -e "\n$(YELLOW) Set url.domain in local /etc/hosts \n"
	echo "127.0.0.1 civi.pontevedra.emergyalabs.com" | sudo tee -a /etc/hosts

down:  ## Stop local stack
	@echo -e "\n$(YELLOW) Stop docker-compose stack \n"
	docker-compose down
	@echo -e "\n$(YELLOW) Remove url.domain from local /etc/hosts \n"
	sudo sed -i "/civi.pontevedra.emergyalabs.com/d" /etc/hosts

bootstrap-drupal:  ## Create a new drupal codebase
	@echo -e "\n${YELLOW} Bootstraping drupal \n"
	docker run --user $$(id -u):$$(id -g) --rm -i --tty -v $${PWD}:/app composer:2.1 \
		create-project drupal/recommended-project:$${DRUPAL_VERSION:-9} src --ignore-platform-reqs
		
bootstrap-civi: ## Add civi requirements to drupal codebase
	docker run --user $$(id -u):$$(id -g) --rm -i --tty -v $${PWD}/src:/app composer:2.1 bash -c "\
		composer require --ignore-platform-reqs --update-with-all-dependencies drush/drush:~10 &&\
		composer config extra.enable-patching true &&\
		composer config minimum-stability dev &&\
		composer require --ignore-platform-reqs civicrm/civicrm-asset-plugin:'~1.1' &&\
		COMPOSER_COMPILE=all composer require --ignore-platform-reqs --no-interaction --update-with-all-dependencies civicrm/civicrm-{core,packages,drupal-8}:'~5.41'"

install-drupal-civi: ## Install drupal and civi-crm
	docker-compose exec php bash -c "drush site:install -y && cv core:install --no-interaction --force --cms-base-url=\"https://civi.pontevedra.emergyalabs.com\""

mysql-dump: ## Mysql dump
	# [TODO] do not dump cache tables for drupal &6 civi
	#docker-compose exec db bash -c 'exec mysqldump $$MYSQL_DATABASE -u $$MYSQL_USER -p$$MYSQL_PASSWORD' > data/initial.sql
	docker-compose exec php bash -c "drush sql-dump --skip-tables-list='cache_*'" > data/initial.sql