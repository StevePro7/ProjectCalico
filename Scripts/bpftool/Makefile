# Shortcust targets
default: image

## Build binary for current platform
all: image-all
###############################################################################
# Both native and cross architecture builds are supported.
# The target architecture is select by setting the ARCH variable.
# When ARCH is undefined it is set to the detected host architecture.
# When ARCH differs from the host architecture a crossbuild will be performed.
ARCHES = amd64 arm64 armv7 ppc64le

# BUILDARCH is the host architecture
# ARCH is the target architecture
# we need to keep track of them separately
BUILDARCH ?= $(shell uname -m)

# canonicalized names for host architecture
ifeq ($(BUILDARCH),aarch64)
        BUILDARCH=arm64
endif
ifeq ($(BUILDARCH),x86_64)
        BUILDARCH=amd64
endif
ifeq ($(BUILDARCH),armv7l)
        BUILDARCH=armv7
endif

# unless otherwise set, I am building for my own architecture, i.e. not cross-compiling
ARCH ?= $(BUILDARCH)

# canonicalized names for target architecture
ifeq ($(ARCH),aarch64)
        override ARCH=arm64
endif
ifeq ($(ARCH),x86_64)
        override ARCH=amd64
endif
ifeq ($(ARCH),armv7l)
        ARCH=armv7
endif

###############################################################################
DOCKERFILE ?= Dockerfile.$(ARCH)
VERSION ?= v5.3
DEFAULTORG ?= calico
DEFAULTIMAGE ?= $(DEFAULTORG)/bpftool:$(VERSION)
ARCHIMAGE ?= $(DEFAULTIMAGE)-$(ARCH)
BPFTOOLIMAGE ?= $(DEFAULTIMAGE)-$(BUILDARCH)
KERNELREF ?= $(VERSION)
KERNELREPO ?= git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git

MANIFEST_TOOL_VERSION := v0.7.0
MANIFEST_TOOL_DIR := $(shell mktemp -d)
export PATH := $(MANIFEST_TOOL_DIR):$(PATH)

space :=
space +=
comma := ,
prefix_linux = $(addprefix linux/,$(strip $(subst armv,arm/v,$1)))
join_platforms = $(subst $(space),$(comma),$(call prefix_linux,$(strip $1)))

# Check if the docker daemon is running in experimental mode (to get the --squash flag)
DOCKER_EXPERIMENTAL=$(shell docker version -f '{{ .Server.Experimental }}')
DOCKER_BUILD_ARGS=
ifeq ($(DOCKER_EXPERIMENTAL),true)
        override DOCKER_BUILD_ARGS=--squash
endif

###############################################################################
# Building the image
###############################################################################
image: $(DEFAULTORG)/bpftool
$(DEFAULTORG)/bpftool: register
	# Make sure we re-pull the base image to pick up security fixes.
	# Limit the build to use only one CPU, This helps to work around qemu bugs such as https://bugs.launchpad.net/qemu/+bug/1098729
	docker build $(DOCKER_BUILD_ARGS) --build-arg KERNEL_REF=$(KERNELREF) --build-arg KERNEL_REPO=$(KERNELREPO) --cpuset-cpus 0 --pull -t $(ARCHIMAGE) -f $(DOCKERFILE) .


# Enable binfmt adding support for miscellaneous binary formats.
.PHONY: register
register:
ifeq ($(ARCH),amd64)
	docker run --rm --privileged multiarch/qemu-user-static:register --reset
endif