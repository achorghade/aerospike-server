# Aerospike Server
# Makefile

DEPTH = ../..
include $(DEPTH)/make_in/Makefile.in

# Use SystemTap?  [By default, no.]
USE_SYSTEMTAP = 0

ifeq ($(USE_SYSTEMTAP),1)
CFLAGS +=	-DUSE_SYSTEMTAP
endif

ifeq ($(USE_SYSTEMTAP),1)
SYSTEMTAP_PROBES_D = base/probes.d
SYSTEMTAP_PROBES_H = $(GEN_DIR)/probes.h
SYSTEMTAP_PROBES_O = $(OBJECT_DIR)/probes.o
endif

ifeq ($(USE_EE),1)
  include $(EEREPO)/as/make_in/Makefile.vars
endif

BASE_HEADERS += aggr.h asm.h batch.h cdt.h cfg.h cluster_config.h datamodel.h index.h job_manager.h json_init.h
BASE_HEADERS += ldt.h ldt_aerospike.h ldt_record.h monitor.h packet_compression.h
BASE_HEADERS += particle.h particle_blob.h particle_integer.h
BASE_HEADERS += proto.h rec_props.h scan.h secondary_index.h security.h security_config.h system_metadata.h
BASE_HEADERS += thr_batch.h thr_info.h thr_proxy.h thr_rw_internal.h thr_sindex.h
BASE_HEADERS += thr_tsvc.h thr_write.h transaction.h transaction_policy.h
BASE_HEADERS += udf_aerospike.h udf_arglist.h udf_cask.h
BASE_HEADERS += udf_memtracker.h udf_record.h udf_rw.h udf_timer.h
BASE_HEADERS += write_request.h xdr_serverside.h

BASE_SOURCES += aggr.c as.c asm.c batch.c bin.c cdt.c cfg.c cluster_config.c index.c job_manager.c json_init.c
BASE_SOURCES += ldt.c ldt_record.c ldt_aerospike.c monitor.c namespace.c packet_compression.c
BASE_SOURCES += particle.c particle_blob.c particle_float.c particle_geojson.c particle_integer.c
BASE_SOURCES += particle_list.c particle_map.c particle_string.c
BASE_SOURCES += proto.c rec_props.c record.c scan.c signal.c secondary_index.c system_metadata.c
BASE_SOURCES += thr_batch.c thr_demarshal.c thr_info.c thr_info_port.c thr_nsup.c thr_proxy.c
BASE_SOURCES += thr_query.c thr_rw.c thr_sindex.c thr_tsvc.c transaction.c
BASE_SOURCES += udf_aerospike.c udf_arglist.c udf_cask.c
BASE_SOURCES += udf_memtracker.c udf_record.c udf_rw.c udf_timer.c
ifneq ($(USE_XDR),1)
  BASE_SOURCES += xdr_serverside_stubs.c
endif
ifneq ($(USE_WARM),1)
  BASE_SOURCES += namespace_ce.c
endif
ifneq ($(USE_SEC),1)
  BASE_SOURCES += security_ce.c
endif

FABRIC_HEADERS += hb.h fabric.h migrate.h paxos.h
FABRIC_SOURCES += hb.c fabric.c migrate.c partition.c paxos.c

GEOSPATIAL_HEADERS += geospatial.h
GEOSPATIAL_SOURCES += geospatial.cc geojson.cc

STORAGE_HEADERS += storage.h drv_ssd.h
STORAGE_SOURCES += storage.c drv_kv.c drv_memory.c drv_ssd.c
ifneq ($(USE_WARM),1)
  STORAGE_SOURCES += drv_ssd_ce.c
endif

HEADERS = $(BASE_HEADERS:%=base/%) $(FABRIC_HEADERS:%=fabric/%) $(STORAGE_HEADERS:%=storage/%) $(GEOSPATIAL_HEADERS:%=geospatial/%)
SOURCES = $(BASE_SOURCES:%=base/%) $(FABRIC_SOURCES:%=fabric/%) $(STORAGE_SOURCES:%=storage/%) $(GEOSPATIAL_SOURCES:%=geospatial/%)

SERVER = $(BIN_DIR)/asd

INCLUDES += $(INCLUDE_DIR:%=-I%) -I$(XDR_INCLUDE_DIR)
ifeq ($(USE_KV),1)
  INCLUDES += -I$(KV_INCLUDE_DIR)
endif
INCLUDES += -I$(CF)/include
INCLUDES += -I$(AI)/include
INCLUDES += -I$(COMMON)/target/$(PLATFORM)/include
INCLUDES += -I$(MOD_LUA)/target/$(PLATFORM)/include
INCLUDES += -I$(JANSSON)/src
INCLUDES += -I$(S2)

ifeq ($(USE_LUAJIT),1)
  INCLUDES += -I$(LUAJIT)/src
else
  INCLUDE_LUA_5_1 = /usr/include/lua5.1
  ifneq ($(wildcard $(INCLUDE_LUA_5_1)),)
    INCLUDES += -I$(INCLUDE_LUA_5_1)
    LUA_SUFFIX = 5.1
  endif
endif

AS_LIBRARIES += $(LIBRARY_DIR)/libcf.a
AS_LIBRARIES += $(LIBRARY_DIR)/libai.a
AS_LIBRARIES += $(MOD_LUA)/target/$(PLATFORM)/lib/libmod_lua.a
AS_LIBRARIES += $(COMMON)/target/$(PLATFORM)/lib/libaerospike-common.a
AS_LIBRARIES += $(LIBRARY_DIR)/libxdrconfig.a

ifeq ($(DOPROFILE),1)
  LIBRARIES += -pg -fprofile-arcs -lgcov
endif

# Add either the LuaJIT or Lua library
ifeq ($(USE_LUAJIT),1)
  ifeq ($(LD_LUAJIT),static)
    AS_LIBRARIES += $(LUAJIT)/src/libluajit.a
  else
    LIBRARIES += -L$(LUAJIT)/src -lluajit
  endif
else
  ifeq ($(LD_LUA),static)
    # Find and add the static Lua library.
    AS_LIBRARIES += $(or \
      $(wildcard /usr/local/lib/liblua.a), \
      $(wildcard /usr/lib64/liblua$(LUA_SUFFIX).a), \
      $(wildcard /usr/lib/x86_64-linux-gnu/liblua$(LUA_SUFFIX).a), \
      $(wildcard /usr/lib/liblua.a), \
      $(wildcard /usr/lib/powerpc64le-linux-gnu/liblua.a), \
      $(error Cannot find "liblua.a"))
  else
    LIBRARIES += -llua$(LUA_SUFFIX)
  endif
endif

ifeq ($(LD_JANSSON),static)
  AS_LIBRARIES += $(JANSSON)/src/.libs/libjansson.a
else
  LIBRARIES += -L$(JANSSON)/src/.libs -ljansson
endif

LIBRARIES += -L$(S2) -ls2 -ls2cellid -lgoogle-strings -lgoogle-base \
			-lgoogle-util-coding -lgoogle-util-math -lstdc++

LIBRARIES := $(AS_LIBRARIES) $(LIBRARIES)

ifneq ($(PREPRO),1)
  ifneq ($(MEXP_PHASE),1)
    AS_LIB_DEPS = $(AS_LIBRARIES)
  endif
endif

ifeq ($(USE_KV),1)
  LIBRARIES += -lvsl -lvsldp
  CFLAGS += -DUSE_KV
endif

OBJECTS.c = $(SOURCES:%.c=$(OBJECT_DIR)/%.o) $(VERSION_OBJ) $(SYSTEMTAP_PROBES_O)
OBJECTS = $(OBJECTS.c:%.cc=$(OBJECT_DIR)/%.o)
DEPENDENCIES = $(OBJECTS:%.o=%.d)
MEXP_SOURCES = $(SOURCES:%=$(MEXP_DIR)/%)
PREPROS = $(OBJECTS:%=%$(PREPRO_SUFFIX))

.PHONY: all
all: $(SYSTEMTAP_PROBES_H) $(SERVER)

.PHONY: clean
clean:
	$(RM) $(OBJECTS) $(SERVER){,.stripped}
	$(RM) $(DEPENDENCIES)
	$(RM) $(MEXP_SOURCES) $(PREPROS)

# Emacs syntax check target.CHK_SOURCES is set by emacs to the files being edited.
.PHONY: check-syntax
check-syntax:
	$(CC) -Wall -Wextra -pedantic -fsyntax-only $(CHK_SOURCES)

$(SERVER): $(OBJECTS) $(AS_LIB_DEPS)
ifneq ($(PREPRO),1)
  ifneq ($(MEXP_PHASE),1)
	$(LINK.c) -o $(SERVER) $(OBJECTS) $(LIBRARIES)
  endif
endif

include $(DEPTH)/make_in/Makefile.targets

# Ignore S2 induced warnings
S2_WNO = -Wno-unused-local-typedefs -Wno-deprecated -Wno-sign-compare
$(OBJECT_DIR)/geospatial/%.o: CXXFLAGS += $(S2_WNO)
$(OBJECT_DIR)/geospatial/%.o: CFLAGS := $(filter-out -std=gnu99,$(CFLAGS))

ifeq ($(USE_SYSTEMTAP),1)
$(SYSTEMTAP_PROBES_H):	$(SYSTEMTAP_PROBES_D)
	dtrace -h -s $< -o $@

$(SYSTEMTAP_PROBES_O):	$(SYSTEMTAP_PROBES_D)
	dtrace -G -s $< -o $@
endif
