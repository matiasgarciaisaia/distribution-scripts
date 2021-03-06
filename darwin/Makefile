# Recipes for this Makefile

## Build everything
##   $ make

CRYSTAL_VERSION ?=                 ## How the binaries should be branded
CRYSTAL_SHA1 ?= $(CRYSTAL_VERSION) ## Git tag/branch/sha1 to checkout and build source
PACKAGE_ITERATION ?= 1
FORCE_GIT_TAGGED ?= 1              ## Require build to be based on git tag/branch

PREVIOUS_CRYSTAL_RELEASE_DARWIN_TARGZ ?= ## url to crystal-{version}-{package}-darwin-x86_64.tar.gz

OUTPUT_DIR = build

# mimics the tgz_package.rb version mangling
DARWIN_PREFIX = crystal-$(CRYSTAL_VERSION)-$(PACKAGE_ITERATION)
DARWIN_NAME = $(DARWIN_PREFIX)-darwin-x86_64.tar.gz
DARWIN_PKG_NAME = $(DARWIN_PREFIX).pkg

.PHONY: all
all: darwin-previous $(OUTPUT_DIR)/$(DARWIN_NAME) $(OUTPUT_DIR)/$(DARWIN_PKG_NAME) ## Build compressed omnibus and distribution packages [default]

.PHONY: help
help: ## Show this help
	@echo
	@printf '\033[34mtargets:\033[0m\n'
	@grep -hE '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) |\
		sort |\
		awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}'
	@echo
	@printf '\033[34moptional variables:\033[0m\n'
	@grep -hE '^[a-zA-Z_-]+ \?=.*?## .*$$' $(MAKEFILE_LIST) |\
		sort |\
		awk 'BEGIN {FS = " \\?=.*?## "}; {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}'
	@echo
	@printf '\033[34mrecipes:\033[0m\n'
	@grep -hE '^##.*$$' $(MAKEFILE_LIST) |\
awk 'BEGIN {FS = "## "}; /^## [a-zA-Z_-]/ {printf "  \033[36m%s\033[0m\n", $$2}; /^##  / {printf "  %s\n", $$2}'

.PHONY: darwin-previous
darwin-previous: $(CURDIR)/../omnibus/crystal-darwin-x86_64 ## download previous crystal darwin release

$(CURDIR)/../omnibus/crystal-darwin-x86_64:
	curl -L -o /tmp/crystal-darwin-x86_64.tar.gz $(PREVIOUS_CRYSTAL_RELEASE_DARWIN_TARGZ) \
	&& tar xfz /tmp/crystal-darwin-x86_64.tar.gz -O "crystal-*/embedded/bin/crystal" > $(CURDIR)/../omnibus/crystal-darwin-x86_64 \
	&& rm /tmp/crystal-darwin-x86_64.tar.gz \
	&& chmod +x $(CURDIR)/../omnibus/crystal-darwin-x86_64

$(OUTPUT_DIR)/$(DARWIN_NAME) $(OUTPUT_DIR)/$(DARWIN_PKG_NAME): ## Build omnibus crystal project
ifeq ($(FORCE_GIT_TAGGED), 0)
	rm -Rf $(CURDIR)/tmp && mkdir -p $(CURDIR)/tmp && cd $(CURDIR)/tmp \
	&& git clone https://github.com/crystal-lang/crystal \
	&& cd crystal \
	&& git checkout $(CRYSTAL_SHA1) \
	&& git checkout -b $(CRYSTAL_VERSION)
endif

	mkdir -p $(OUTPUT_DIR) \
	&& cd ../omnibus \
  && if [ "$(FORCE_GIT_TAGGED)" -eq "0" ]; then export CRYSTAL_SRC=$(CURDIR)/tmp/crystal/.git; fi \
	&& bundle exec omnibus clean crystal shards \
  && bundle exec omnibus build crystal \
  && cp ./pkg/$(DARWIN_NAME) ./pkg/$(DARWIN_PKG_NAME) $(CURDIR)/$(OUTPUT_DIR)/

.PHONY: clean
clean: ## Clean up build directory
	rm -Rf $(OUTPUT_DIR)
	rm -Rf $(CURDIR)/tmp
	rm -Rf $(CURDIR)/../omnibus/pkg/crystal-*
	rm -Rf $(CURDIR)/../omnibus/pkg/version-*
	rm -Rf $(CURDIR)/../omnibus/crystal-darwin-x86_64
	rm -Rf /var/cache/omnibus/*
	rm -Rf /opt/crystal/*
