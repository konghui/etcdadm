# Copyright 2020 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

STABLE_DOCKER_REGISTRY     := $(shell tools/get_workspace_status.sh | grep STABLE_DOCKER_REGISTRY | cut -d ' ' -f 2)
STABLE_DOCKER_IMAGE_PREFIX := $(shell tools/get_workspace_status.sh | grep STABLE_DOCKER_IMAGE_PREFIX | cut -d ' ' -f 2)
STABLE_DOCKER_TAG          := $(shell tools/get_workspace_status.sh | grep STABLE_DOCKER_TAG | cut -d ' ' -f 2)
IMAGE_BASE                 := $(STABLE_DOCKER_REGISTRY)/$(STABLE_DOCKER_IMAGE_PREFIX)

BAZEL_FLAGS=--features=pure --platforms=@io_bazel_rules_go//go/toolchain:linux_amd64

.PHONY: all
all: test

.PHONY: test
test:
	bazel test //... --test_output=streamed

.PHONY: stress-test
stress-test:
	bazel test //... --test_output=streamed --runs_per_test=10

.PHONY: gofmt
gofmt:
	gofmt -w -s cmd/ pkg/

.PHONY: goimports
goimports:
	goimports -w cmd/ pkg/ test/

.PHONY: push-etcd-manager
push-etcd-manager:
	bazel run ${BAZEL_FLAGS} --platforms=@io_bazel_rules_go//go/toolchain:linux_amd64 //images:push-etcd-manager
	bazel run ${BAZEL_FLAGS} --platforms=@io_bazel_rules_go//go/toolchain:linux_arm64 //images:push-etcd-manager

.PHONY: push-etcd-manager-manifest
push-etcd-manager-manifest:
	DOCKER_CLI_EXPERIMENTAL=enabled docker manifest create \
		$(IMAGE_BASE)etcd-manager:$(STABLE_DOCKER_TAG) \
		$(IMAGE_BASE)etcd-manager:$(STABLE_DOCKER_TAG)-amd64 \
		$(IMAGE_BASE)etcd-manager:$(STABLE_DOCKER_TAG)-arm64
	DOCKER_CLI_EXPERIMENTAL=enabled docker manifest push --purge \
		$(IMAGE_BASE)etcd-manager:$(STABLE_DOCKER_TAG)

.PHONY: push-etcd-dump
push-etcd-dump:
	bazel run ${BAZEL_FLAGS} --platforms=@io_bazel_rules_go//go/toolchain:linux_amd64 //images:push-etcd-dump
	bazel run ${BAZEL_FLAGS} --platforms=@io_bazel_rules_go//go/toolchain:linux_arm64 //images:push-etcd-dump

.PHONY: push-etcd-dump-manifest
push-etcd-dump-manifest:
	DOCKER_CLI_EXPERIMENTAL=enabled docker manifest create \
		$(IMAGE_BASE)etcd-dump:$(STABLE_DOCKER_TAG) \
		$(IMAGE_BASE)etcd-dump:$(STABLE_DOCKER_TAG)-amd64 \
		$(IMAGE_BASE)etcd-dump:$(STABLE_DOCKER_TAG)-arm64
	DOCKER_CLI_EXPERIMENTAL=enabled docker manifest push --purge \
		$(IMAGE_BASE)etcd-dump:$(STABLE_DOCKER_TAG)

.PHONY: push-etcd-backup
push-etcd-backup:
	bazel run ${BAZEL_FLAGS} --platforms=@io_bazel_rules_go//go/toolchain:linux_amd64 //images:push-etcd-backup
	bazel run ${BAZEL_FLAGS} --platforms=@io_bazel_rules_go//go/toolchain:linux_arm64 //images:push-etcd-backup

.PHONY: push-etcd-backup-manifest
push-etcd-backup-manifest:
	DOCKER_CLI_EXPERIMENTAL=enabled docker manifest create \
		$(IMAGE_BASE)etcd-backup:$(STABLE_DOCKER_TAG) \
		$(IMAGE_BASE)etcd-backup:$(STABLE_DOCKER_TAG)-amd64 \
		$(IMAGE_BASE)etcd-backup:$(STABLE_DOCKER_TAG)-arm64
	DOCKER_CLI_EXPERIMENTAL=enabled docker manifest push --purge \
		$(IMAGE_BASE)etcd-backup:$(STABLE_DOCKER_TAG)

.PHONY: push-images
push-images: push-etcd-manager push-etcd-dump push-etcd-backup
	echo "pushed images"

.PHONY: push-manifests
push-manifests: push-etcd-manager-manifest push-etcd-dump-manifest push-etcd-backup-manifest
	echo "pushed manifests"

.PHONY: push
push: push-images push-manifests

.PHONY: gazelle
gazelle:
	bazel run //:gazelle -- fix
	cd tools/deb-tools && bazel run //:gazelle -- fix
	git checkout -- vendor
	rm -f vendor/github.com/coreos/etcd/cmd/etcd
	#rm vendor/github.com/golang/protobuf/protoc-gen-go/testdata/multi/BUILD.bazel

.PHONY: dep-ensure
dep-ensure:
	echo "'make dep-ensure' has been replaced by 'make vendor'"
	exit 1

.PHONY: vendor
vendor:
	go mod tidy
	go mod vendor
	find vendor/ -name "BUILD" -delete
	find vendor/ -name "BUILD.bazel" -delete
	bazel run //:gazelle
	make -C tools/deb-tools vendor

.PHONY: staticcheck-all
staticcheck-all:
	go list ./... | xargs go run honnef.co/go/tools/cmd/staticcheck@v0.2.1

# staticcheck-working is the subset of packages that we have cleaned up
# We gradually want to sync up staticcheck-all with staticcheck-working
.PHONY: staticcheck-working
staticcheck-working:
	go list ./... | grep -v "etcd-manager/pkg/[cepv]" | xargs go run honnef.co/go/tools/cmd/staticcheck@v0.2.1
