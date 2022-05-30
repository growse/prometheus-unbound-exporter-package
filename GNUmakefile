DEBNAME := prometheus-unbound-exporter
APP_REMOTE := github.com/letsencrypt/unbound_exporter
VERSION := v0.4.1
APPDESCRIPTION := Exporter for unbound metrics
APPURL := https://github.com/letsencrypt/unbound_exporter
ARCH := amd64
GO_BUILD_SOURCE := .

# Setup
BUILD_NUMBER ?= 0
DEBVERSION := $(VERSION:v%=%)-$(BUILD_NUMBER)
GOPATH := $(abspath gopath)
APPHOME := $(GOPATH)/src/$(APP_REMOTE)

# Let's map from go architectures to deb architectures, because they're not the same!
DEB_amd64_ARCH := amd64

# Version info for binaries
CGO_ENABLED := 0
VPREFIX := github.com/prometheus/common/version

GO_LDFLAGS = -s -w --extldflags '-static'
DYN_GO_FLAGS = -ldflags "$(GO_LDFLAGS)" -tags netgo

.EXPORT_ALL_VARIABLES:

.PHONY: package
package: $(addsuffix .deb, $(addprefix $(DEBNAME)_$(DEBVERSION)_, $(foreach a, $(ARCH), $(a))))

.PHONY: build
build: $(addprefix $(APPHOME)/dist/$(DEBNAME)_linux_, $(foreach a, $(ARCH), $(a)))

.PHONY: checkout
checkout: $(APPHOME)

$(GOPATH):
	mkdir $(GOPATH)

$(APPHOME): $(GOPATH)
	git clone https://$(APP_REMOTE) $(APPHOME)
	cd $(APPHOME) && git checkout $(VERSION)

$(APPHOME)/dist/$(DEBNAME)_linux_%: $(APPHOME)
	$(eval GIT_REVISION := $(shell cd $(APPHOME) && git rev-parse --short HEAD))
	$(eval GIT_BRANCH := $(shell cd $(APPHOME) && git rev-parse --abbrev-ref HEAD))
	$(eval IMAGE_TAG := $(shell cd $(APPHOME) && git describe --exact-match))
	cd $(APPHOME) && \
	GOOS=linux GOARCH=$* go build $(DYN_GO_FLAGS) -o dist/$(DEBNAME)_linux_$* $(GO_BUILD_SOURCE)
	upx $@

$(DEBNAME)_$(DEBVERSION)_%.deb: $(APPHOME)/dist/$(DEBNAME)_linux_%
	chmod +x $<
	bundle exec fpm \
	-s dir \
	-t deb \
	--license "Apache 2.0" \
	--deb-priority optional \
	--maintainer github@growse.com \
	--vendor ISRG \
	-n $(DEBNAME) \
	--description \
	"$(APPDESCRIPTION)" \
	--url $(APPURL) \
	--prefix / \
	-a $(DEB_$*_ARCH) \
	-v $(DEBVERSION) \
	--deb-systemd prometheus-unbound-exporter.service \
	prometheus-unbound-exporter.defaults=/etc/default/prometheus-unbound-exporter \
	$<=/usr/sbin/unbound_exporter

.PHONY: clean
clean:
	rm -f *.deb
	rm -rf $(GOPATH)
