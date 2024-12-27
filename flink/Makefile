SHELL=/bin/bash
NAME               := flink
FLINK_VERSION      := 1.20.0
BASE_IMAGE         := apache/flink:${FLINK_VERSION}
REVISION           := $(shell git rev-parse --short HEAD)
ORIGIN             := $(shell git remote get-url origin | sed -e 's/^.*@//g')
IMAGE_VERSION      := 1
RELEASE_TAGS       := $(IMAGE_VERSION) $(FLINK_VERSION)-$(IMAGE_VERSION)
RELEASE_TEST_TAGS  := devel-PR$(CHANGE_ID)
REGISTRY           := docker.io
USER               := yassan
PROXY              := ${HTTP_PROXY}
DOCKER_FILE        ?= Dockerfile
BUILD_OPS          ?= --progress=plain

.PHONY: build
build:
	if [ -n "${HTTP_PROXY}" ]; then \
		docker build $(BUILD_OPS) \
			--build-arg flink_version=$(FLINK_VERSION) \
			--build-arg base_image=$(BASE_IMAGE) \
			--build-arg git_revision=$(REVISION) \
			--build-arg git_origin=$(ORIGIN) \
			--build-arg image_name=$(REGISTRY)/$(USER)/$(NAME) \
			--build-arg http_proxy=$(PROXY) \
			--build-arg https_proxy=$(PROXY) \
			--build-arg no_proxy="$(NO_PROXY)" \
			$(addprefix -t $(REGISTRY)/$(USER)/$(NAME):,$(TAGS)) \
			-f ./docker/$(DOCKER_FILE) ./docker/ ; \
	else \
		docker build $(BUILD_OPS) \
			--build-arg flink_version=$(FLINK_VERSION) \
			--build-arg base_image=$(BASE_IMAGE) \
			--build-arg git_revision=$(REVISION) \
			--build-arg git_origin=$(ORIGIN) \
			--build-arg image_name=$(REGISTRY)/$(USER)/$(NAME) \
			$(addprefix -t $(REGISTRY)/$(USER)/$(NAME):,$(TAGS)) \
			-f ./docker/$(DOCKER_FILE) ./docker/ ; \
	fi ;

.PHONY: push
push:
	@for TAG in $(TAGS); do\
		docker push $(REGISTRY)/$(USER)/$(NAME):$$TAG ;\
	done

.PHONY: release
release:
	@make build TAGS="$(RELEASE_TAGS)"
	@make push TAGS="$(RELEASE_TAGS)"
	@make clean TAGS="$(RELEASE_TAGS)"

.PHONY: clean
clean:
	@for TAG in $(TAGS); do \
		if [ -n "$$(docker images -q $(REGISTRY)/$(USER)/$(NAME):$$TAG)" ]; then \
			docker image rm $(REGISTRY)/$(USER)/$(NAME):$$TAG ; \
		fi \
	done ;

.PHONY: release-test
release-test:
	env
	@make build TAGS="$(RELEASE_TEST_TAGS)"
	@make push TAGS="$(RELEASE_TEST_TAGS)"
	@make clean TAGS="$(RELEASE_TEST_TAGS)"

.PHONY: test
test:
	make clean TAGS="test"

	make build TAGS="test"

	docker run --rm $(REGISTRY)/$(USER)/$(NAME):test date ;
	docker run --rm $(REGISTRY)/$(USER)/$(NAME):test bash -c "[ -f /opt/flink/plugins/s3/flink-s3-fs-hadoop-1.20.0.jar ] || { echo \"not exist.\" >&2; exit 1; }" ;
