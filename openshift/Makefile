RELEASE_TAG  ?= $(shell grep FROM Dockerfile.openshift |grep -v ' AS builder' |sed -e 's;.*/;;' -e 's;:.*;;')
OCP_REGISTRY := catalog.redhat.com/software/containers/openshift4
IMAGE_NAME   := ose-cluster-kube-cluster-api-rhel9-operator

BIN_DIR := bin
TOOLS_DIR := tools

KUSTOMIZE := tools/bin/kustomize
ROOT:=$(shell dirname $(realpath ../$(firstword $(MAKEFILE_LIST))))
YQ := $(ROOT)/hack/tools/bin/yq
HELM := $(ROOT)/hack/tools/bin/helm
MAKE := make

$(KUSTOMIZE):
	./tools/ensure-kustomize.sh

.PHONY: ocp-manifests
ocp-manifests: $(KUSTOMIZE) ## Builds openshift specific manifests
	$(KUSTOMIZE) build ../config/default > operator-components.yaml

HELM_CHART_TAG := $(RELEASE_TAG)
RELEASE_DIR := $(ROOT)/out
CHART_DIR := $(RELEASE_DIR)/charts/cluster-api-operator
CHART_PACKAGE_DIR := $(RELEASE_DIR)/package
CHART_VALUES_FILE := $(CHART_DIR)/values.yaml
CHART_TEMPLATE_FILE := $(CHART_DIR)/templates/deployment.yaml

.PHONY: release-chart
release-chart:
	$(MAKE) -C .. release-chart
	$(MAKE) -C .. set-manifest-image-chart \
		MANIFEST_IMG=$(REGISTRY)/$(IMAGE_NAME) MANIFEST_TAG=$(RELEASE_TAG) \
	        TARGET_RESOURCE="$(CHART_VALUES_FILE)"
	$(MAKE) -C .. set-manifest-pull-policy-chart \
	        PULL_POLICY=IfNotPresent \
	        TARGET_RESOURCE="$(CHART_VALUES_FILE)"
	# $(YQ) eval '.core      = "xxx"' $(CHART_VALUES_FILE) -i
	# $(YQ) eval '.bootstrap = "yyy"' $(CHART_VALUES_FILE) -i
	sed -i -e 's;^\([[:space:]]*-[[:space:]]*\)/manager;\1/bin/cluster-api-operator;' $(CHART_TEMPLATE_FILE)
	$(HELM) package $(CHART_DIR) --app-version=$(HELM_CHART_TAG) --version=$(HELM_CHART_TAG) --destination=$(CHART_PACKAGE_DIR)
