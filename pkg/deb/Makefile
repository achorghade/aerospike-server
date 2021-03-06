# Build Aerospike Server ".deb" Distribution.

DEPTH = ../..
include $(DEPTH)/make_in/Makefile.vars

PKG = $(realpath $(DEPTH)/pkg)
SOURCE_ROOT = $(PKG)/dist
BUILD_ROOT = $(SOURCE_ROOT)/BUILD
OPT_AS = $(BUILD_ROOT)/opt/aerospike

REV = $(shell $(DEPTH)/build/version)
OS = $(shell $(DEPTH)/build/os_version)
SIZE = $(shell du -k $(BIN_DIR)/asd | cut -f1)
DEPS = ", logrotate"

ifeq ($(OS),debian8)
  USE_SYSTEMD = 1
  DEPS =
endif

DEB = $(PKG)/packages/aerospike-server-$(EDITION)-$(REV).$(OS).x86_64.deb

ifeq ($(USE_XDR),1)
all:	dist-xdr package clean
else
all:	dist package clean
endif

.PHONY: dist
dist:
	install -d $(BUILD_ROOT)/DEBIAN
	install -d $(BUILD_ROOT)/etc/aerospike
	install -d $(BUILD_ROOT)/etc/aerospike/sample
ifeq ($(USE_SYSTEMD),1)
	install -d $(BUILD_ROOT)/usr/lib/systemd/system
	install -d $(BUILD_ROOT)/etc/systemd/system/aerospike.service.d
else
	install -d $(BUILD_ROOT)/etc/init.d
	install -d $(BUILD_ROOT)/etc/logrotate.d
	install -d $(BUILD_ROOT)/var/log/aerospike
	install -d $(BUILD_ROOT)/var/run/aerospike
endif
	install -d $(BUILD_ROOT)/usr/bin

	install -pm 644 $(PKG)/deb/conffiles $(BUILD_ROOT)/DEBIAN
ifeq ($(EDITION),community)
	cat $(PKG)/deb/conffiles.telemetry >> $(BUILD_ROOT)/DEBIAN/conffiles
endif
	install -pm 755 $(PKG)/deb/postinst.server $(BUILD_ROOT)/DEBIAN/postinst

	install -pm 755 $(BIN_DIR)/asd $(BUILD_ROOT)/usr/bin/asd
ifeq ($(USE_SYSTEMD),1)
	install -pm 755 $(DEPTH)/tools/bin/asd-coldstart $(BUILD_ROOT)/usr/bin/asd-coldstart
endif
	install -pm 755 $(DEPTH)/tools/citrus2aero/upgrade2to3 $(BUILD_ROOT)/usr/bin/asmigrate2to3
	install -pm 755 $(DEPTH)/tools/fixownership/fixownership.py $(BUILD_ROOT)/usr/bin/asfixownership

ifeq ($(USE_SYSTEMD),1)
	install -pm 644 $(DEPTH)/as/etc/aerospike_systemd.conf $(BUILD_ROOT)/etc/aerospike/aerospike.conf
else
	install -pm 644 $(DEPTH)/as/etc/aerospike.conf $(BUILD_ROOT)/etc/aerospike
endif
	cat $(DEPTH)/as/etc/README.sample.conf.md > $(BUILD_ROOT)/etc/aerospike/sample/README.md
	install -pm 644 $(DEPTH)/as/etc/aerospike_ssd.conf $(BUILD_ROOT)/etc/aerospike/sample
	install -pm 644 $(DEPTH)/as/etc/aerospike_mesh.conf $(BUILD_ROOT)/etc/aerospike/sample

ifeq ($(USE_SYSTEMD),1)
	cat $(DEPTH)/as/etc/aerospike.service.head >> $(PKG)/deb/aerospike.service
  ifeq ($(EDITION),community)
	cat $(DEPTH)/as/etc/aerospike.service.telemetry >> $(PKG)/deb/aerospike.service
  endif
	cat $(DEPTH)/as/etc/aerospike.service.tail >> $(PKG)/deb/aerospike.service
	install -p -D -m 644 $(PKG)/deb/aerospike.service $(BUILD_ROOT)/usr/lib/systemd/system/aerospike.service
	install -p -D -m 644 $(DEPTH)/as/etc/aerospike-server.tmpfiles $(BUILD_ROOT)/etc/tmpfiles.d/aerospike.conf
	install -p -D -m 644 $(DEPTH)/as/etc/aerospike-server.sysconfig $(BUILD_ROOT)/etc/sysconfig/aerospike
	install -p -D -m 755 $(DEPTH)/as/etc/asd-systemd-helper $(BUILD_ROOT)/usr/bin/asd-systemd-helper
	install -p -D -m 644 $(DEPTH)/as/etc/aerospike.service.d/* $(BUILD_ROOT)/etc/systemd/system/aerospike.service.d
else
	install -pm 755 $(DEPTH)/as/etc/init-script.deb $(BUILD_ROOT)/etc/init.d/aerospike
	sed -i 's/@EDITION@/$(EDITION)/g' $(BUILD_ROOT)/etc/init.d/aerospike
	install -pm 644 $(DEPTH)/as/etc/logrotate_asd $(BUILD_ROOT)/etc/logrotate.d/aerospike
endif

	install -d $(OPT_AS)/doc
	install -pm 644 $(DEPTH)/LICENSE $(OPT_AS)/doc

ifeq ($(EDITION),community)
  ifeq ($(USE_SYSTEMD),1)
	install -pm 755 $(DEPTH)/as/etc/aerospike_telemetry.service $(BUILD_ROOT)/usr/lib/systemd/system/aerospike_telemetry.service
	install -pm 644 $(DEPTH)/as/etc/aerospike_telemetry.sysconfig $(BUILD_ROOT)/etc/sysconfig/aerospike_telemetry
  else
	install -pm 755 $(DEPTH)/as/etc/init-telemetry-script.deb $(BUILD_ROOT)/etc/init.d/aerospike_telemetry
	install -pm 644 $(DEPTH)/as/etc/logrotate_telemetry $(BUILD_ROOT)/etc/logrotate.d/aerospike_telemetry
  endif
	install -d $(OPT_AS)/telemetry
	install -d $(OPT_AS)/telemetry/phonehome
	install -d $(OPT_AS)/telemetry/daemon
	install -pm 644 $(DEPTH)/as/etc/telemetry.conf $(BUILD_ROOT)/etc/aerospike
	install -pm 644 $(DEPTH)/modules/telemetry/README.md $(OPT_AS)/doc/TELEMETRY.md
	install -pm 755 $(DEPTH)/modules/telemetry/telemetry.py $(OPT_AS)/telemetry
	install -pm 755 $(DEPTH)/modules/telemetry/phonehome/*.py $(OPT_AS)/telemetry/phonehome
	install -pm 755 $(DEPTH)/modules/telemetry/daemon/*.py $(OPT_AS)/telemetry/daemon
endif

	install -d $(OPT_AS)/data
	install -d $(OPT_AS)/smd
	install -d $(OPT_AS)/sys/udf/lua/external
	install -d $(OPT_AS)/sys/udf/lua/ldt
	install -d $(OPT_AS)/usr/udf/lua
	install -pm 644 $(DEPTH)/modules/lua-core/src/*.lua $(OPT_AS)/sys/udf/lua
	install -pm 644 $(DEPTH)/modules/lua-core/src/external/* $(OPT_AS)/sys/udf/lua/external
	install -pm 644 $(DEPTH)/modules/lua-core/src/ldt/* $(OPT_AS)/sys/udf/lua/ldt

ifeq ($(USE_ASM),1)
	install -pm 644 $(DEPTH)/as/etc/initfns $(BUILD_ROOT)/etc/aerospike
	install -d $(OPT_AS)/{bin,lib}
	install -pm 755 $(DEPTH)/as/etc/run-under-gdb $(OPT_AS)/bin
	install -pm 644 $(DEPTH)/modules/asmalloc/lib/asmalloc.jem.so $(OPT_AS)/lib
	install -pm 644 $(DEPTH)/modules/jemalloc/lib/libjemalloc.so $(OPT_AS)/lib
endif

	sed 's/@VERSION@/'$(REV)'/g' < $(PKG)/deb/server-64 > $(BUILD_ROOT)/DEBIAN/control
	sed -i 's/@EDITION@/'$(EDITION)'/g' $(BUILD_ROOT)/DEBIAN/control
	sed -i 's/@SIZE@/'$(SIZE)'/g' $(BUILD_ROOT)/DEBIAN/control
	sed -i 's/@DEPS@/'$(DEPS)'/g' $(BUILD_ROOT)/DEBIAN/control

package:
	fakeroot dpkg-deb --build $(BUILD_ROOT) $(DEB)

clean:
	rm -rf $(SOURCE_ROOT)/*
ifeq ($(USE_SYSTEMD),1)
	rm -rf $(PKG)/deb/aerospike.service
endif

ifeq ($(USE_XDR),1)
  include $(XDR)/make_in/Makefile.deb.in
endif
