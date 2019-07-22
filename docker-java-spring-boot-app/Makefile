.PHONY: all jar image

TAG?=latest
ORG?=tombee
DEV_IMAGE_NAME?=spring-boot-app-dev
REPO?=$(ORG)/spring-boot-app

all: test jar
release: all image push

/var/run/docker.sock:
	$(error You must run your container with "-v /var/run/docker.sock:/var/run/docker.sock")

test:
	@mvn verify

jar:
	@mvn package

image: /var/run/docker.sock
	@docker build --build-arg "VERSION=$$TAG" -t $(REPO):$(TAG) .

push: /var/run/docker.sock
	@docker push $(REPO):$(TAG)
