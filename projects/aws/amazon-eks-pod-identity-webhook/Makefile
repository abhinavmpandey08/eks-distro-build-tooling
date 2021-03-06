REPO?=amazon-eks-pod-identity-webhook
CLONE_URL?=https://github.com/aws/$(REPO).git
GIT_COMMIT?=$(shell cat GIT_COMMIT)

ifeq ("$(CLONE_URL)","")
	$(error No clone url was provided.)
endif

ifeq ("$(REPO)","")
	$(error No repository name was provided.)
endif

ifeq ("$(GIT_COMMIT)","")
	$(error No git commit was provided.)
endif

DEVELOPMENT?=true
AWS_ACCOUNT_ID?=$(shell aws sts get-caller-identity --query Account --output text)
AWS_REGION?=us-west-2

IMAGE_REPO?=$(AWS_ACCOUNT_ID).dkr.ecr.$(AWS_REGION).amazonaws.com
IMAGE_NAME?=eks-distro/amazon-eks-pod-identity-webhook
# This tag is overwritten in the prow job to point to the commit hash
IMAGE_TAG?=latest
IMAGE?=$(IMAGE_REPO)/$(IMAGE_NAME):$(IMAGE_TAG)

.PHONY: clone
clone: clean
	git clone $(CLONE_URL) $(REPO)
	cd $(REPO) && git checkout $(GIT_COMMIT)

.PHONY: local-images
local-images: clone
	# Sleep to wait for buildkitd to start up
	sleep 10
	buildctl \
		build \
		--frontend dockerfile.v0 \
		--opt platform=linux/amd64 \
		--local dockerfile=./$(REPO) \
		--local context=./$(REPO) \
		--output type=oci,oci-mediatypes=true,name=$(IMAGE),dest=/tmp/amazon-eks-pod-identity-webhook.tar

.PHONY: images
images: clone
	# Sleep to wait for buildkitd to start up
	sleep 10
	buildctl \
		build \
		--frontend dockerfile.v0 \
		--opt platform=linux/amd64 \
		--local dockerfile=./$(REPO) \
		--local context=./$(REPO) \
		--output type=image,oci-mediatypes=true,name=$(IMAGE),push=true

.PHONY: docker
docker: clone
	docker build -t $(IMAGE) -f ./$(REPO)/Dockerfile ./$(REPO)

.PHONY: build
build: local-images

.PHONY: release
release: images

.PHONY: clean
clean:
	rm -rf $(REPO)
