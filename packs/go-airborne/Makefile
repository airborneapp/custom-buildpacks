SHELL := /bin/bash
GO := GO15VENDOREXPERIMENT=1 go
NAME := REPLACE_ME_APP_NAME
OS := $(shell uname)
MAIN_GO := ./service/REPLACE_ME_APP_NAME/service.go
ROOT_PACKAGE := $(GIT_PROVIDER)/REPLACE_ME_ORG/$(NAME)
GO_VERSION := $(shell $(GO) version | sed -e 's/^[^0-9.]*\([0-9.]*\).*/\1/')
PACKAGE_DIRS := $(shell $(GO) list ./... | grep -v /vendor/)
PKGS := $(shell go list ./... | grep -v /vendor | grep -v generated)
BUILDFLAGS := ''
CGO_ENABLED = 0
VENDOR_DIR=vendor

all: build

check: fmt build test

build:
	#CGO_ENABLED=$(CGO_ENABLED) $(GO) build -ldflags $(BUILDFLAGS) -o bin/$(NAME) $(MAIN_GO)
	CGO_ENABLED=$(CGO_ENABLED) GOOS=linux GOARCH=amd64 $(GO) build -ldflags $(BUILDFLAGS) -o bin/$(NAME) $(MAIN_GO)

test: 
	CGO_ENABLED=$(CGO_ENABLED) $(GO) test $(PACKAGE_DIRS) -test.v

full: $(PKGS)

install:
	GOBIN=${GOPATH}/bin $(GO) install -ldflags $(BUILDFLAGS) $(MAIN_GO)

fmt:
	@FORMATTED=`$(GO) fmt $(PACKAGE_DIRS)`
	@([[ ! -z "$(FORMATTED)" ]] && printf "Fixed unformatted files:\n$(FORMATTED)") || true

clean:
	rm -rf build release

linux:
	CGO_ENABLED=$(CGO_ENABLED) GOOS=linux GOARCH=amd64 $(GO) build -ldflags $(BUILDFLAGS) -o bin/$(NAME) $(MAIN_GO)

.PHONY: release clean

FGT := $(GOPATH)/bin/fgt
$(FGT):
	go get github.com/GeertJohan/fgt

GOLINT := $(GOPATH)/bin/golint
$(GOLINT):
	go get github.com/golang/lint/golint

$(PKGS): $(GOLINT) $(FGT)
	@echo "LINTING"
	@$(FGT) $(GOLINT) $(GOPATH)/src/$@/*.go
	@echo "VETTING"
	@go vet -v $@
	@echo "TESTING"
	@go test -v $@

.PHONY: lint
lint: vendor | $(PKGS) $(GOLINT) # ❷
	@cd $(BASE) && ret=0 && for pkg in $(PKGS); do \
	    test -z "$$($(GOLINT) $$pkg | tee /dev/stderr)" || ret=1 ; \
	done ; exit $$ret


.PHONY: gen-swagger
gen-swagger:
	@mkdir -p ./pkg/gen/server
	/workspace/source/swagger generate server --target ./pkg/gen/server --name SwaggerRest --spec swagger.yaml

.PHONY: gen
gen:
	@mkdir -p ./pkg/gen
	@swagger generate server --target ./pkg/gen --name SwaggerRest --spec swagger.yaml

.PHONY: run
run:
# Service dockerization is disabled for the sake of simplicity and speed of testing.
#	@docker run -it -p 8080:8080 $(DEMO_SERVICE_NAME)-$(GIT_BRANCH):$(DEMO_SERVICE_VERSION)
    # Starts docker containers for MySql, DynamoDb, and Consul. This is required for local testing
	@docker-compose up -d
	@sleep 3