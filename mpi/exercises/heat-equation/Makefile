# SPDX-FileCopyrightText: 2010 CSC - IT Center for Science Ltd. <www.csc.fi>
#
# SPDX-License-Identifier: MIT

# Build the heat equation exercise or one of its model solutions.
# Usage:
#   make                          # build from cpp/
#   make SOLUTION=solution-sendrecv   # build from cpp/solution-sendrecv/

.DEFAULT_GOAL := all

ifeq ($(COMP),)
COMP=lumi
endif

SOLUTION ?=
SRCDIR   := $(if $(SOLUTION),cpp/$(SOLUTION),cpp)
BUILDDIR := build/$(SRCDIR)

# Include the "heat-shared" Makefile: fills in HEAT_SHARED_CFLAGS and adds rule for building HEAT_COMMON_LIB (static)
HEAT_SHARED_DIR := ../../../heat-shared
# Override before including so heat-shared artifacts land in our build tree. Ensures we always rebuild it separately for each solution/implementation
HEAT_COMMON_LIB := $(BUILDDIR)/heat_common.a
include $(HEAT_SHARED_DIR)/Makefile-common

ifeq ($(COMP),gnu)
CXX=mpicxx
CC=mpicc
CCFLAGS=-g -O3 -march=native -Wall -DNDEBUG
LDFLAGS=
LIBS=
endif

ifeq ($(COMP),intel)
CXX=mpicxx
CC=mpicc
CCFLAGS=-g -O3 -xHost -DNDEBUG
LDFLAGS=
LIBS=
endif

ifeq ($(COMP),lumi)
CXX=CC
CC=cc
CCFLAGS=-O3 -Wall
LDFLAGS=
LIBS=
endif

vpath %.cpp $(SRCDIR)
vpath %.hpp $(SRCDIR)

EXE  = heat_mpi
OBJS = $(addprefix $(BUILDDIR)/,main.o heat.o core.o setup.o io.o utilities.o)
# Dependency tracking
DEPS = $(OBJS:.o=.d)

all: $(EXE)

$(BUILDDIR):
	mkdir -p $@

$(EXE): $(OBJS) $(HEAT_COMMON_LIB)
	$(CXX) $(CCFLAGS) $(OBJS) $(HEAT_COMMON_LIB) -o $@ $(LDFLAGS) $(LIBS)

# The -MMD -MP flags instruct the compiler to generate file dependency graphs,
# so that on consecutive builds we only need to rebuild objects that actually changed.

$(BUILDDIR)/%.o: %.cpp | $(BUILDDIR)
	$(CXX) $(CCFLAGS) $(HEAT_SHARED_CFLAGS) -I$(SRCDIR) -MMD -MP -c $< -o $@

$(BUILDDIR)/%.o: %.c | $(BUILDDIR)
	$(CC) $(CCFLAGS) $(HEAT_SHARED_CFLAGS) -I$(SRCDIR) -MMD -MP -c $< -o $@

-include $(DEPS)

.PHONY: clean
clean: heat-common-clean
	-/bin/rm -rf build $(EXE)
