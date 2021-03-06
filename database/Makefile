include ../includes.mk

SHELL_SCRIPTS = $(shell find "." -name '*.sh') $(wildcard bin/*)

COMPONENT = database
IMAGE = $(IMAGE_PREFIX)$(COMPONENT):$(BUILD_TAG)
DEV_IMAGE = $(REGISTRY)$(IMAGE)

build: check-docker
	docker build -t $(IMAGE) .

clean: check-docker check-registry
	docker rmi $(IMAGE)

full-clean: check-docker check-registry
	docker images -q $(IMAGE_PREFIX)$(COMPONENT) | xargs docker rmi -f

install: check-deisctl
	deisctl install $(COMPONENT)

uninstall: check-deisctl
	deisctl uninstall $(COMPONENT)

start: check-deisctl
	deisctl start $(COMPONENT)

stop: check-deisctl
	deisctl stop $(COMPONENT)

restart: stop start

run: install start

dev-release: push set-image

push: check-registry
	docker tag -f $(IMAGE) $(DEV_IMAGE)
	docker push $(DEV_IMAGE)

set-image: check-deisctl
	deisctl config $(COMPONENT) set image=$(DEV_IMAGE)

release:
	docker push $(IMAGE)

deploy: build dev-release restart

test: test-style test-unit test-functional

test-unit:
	@echo no unit tests

test-style:
	shellcheck $(SHELL_SCRIPTS)

test-functional:
	@$(MAKE) -C ../tests/ mock-store
	@$(MAKE) -C ../tests/ test-etcd
	GOPATH=`cd ../tests/ && godep path`:$(GOPATH) go test -v ./tests/...
