VERSION = 1.36
IMAGE_NAME ?= wonkyto/telegraf:$(VERSION)

build-pi:
	docker buildx build --platform linux/arm64 -t $(IMAGE_NAME) --push .
build-all:
	docker buildx build --platform linux/amd64,linux/arm64 -t $(IMAGE_NAME) --push .
