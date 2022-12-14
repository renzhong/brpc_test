# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

NEED_GPERFTOOLS=1
BRPC_PATH = ../../
include $(BRPC_PATH)/config.mk
# Notes on the flags:
# 1. Added -fno-omit-frame-pointer: perf/tcmalloc-profiler use frame pointers by default
CXXFLAGS = $(CPPFLAGS) -std=c++0x -g -DNDEBUG -O2 -pipe -W -Wall -Wno-unused-parameter -fPIC -fno-omit-frame-pointer
ifeq ($(NEED_GPERFTOOLS), 1)
	CXXFLAGS+=-DBRPC_ENABLE_CPU_PROFILER
endif
HDRS+=$(BRPC_PATH)/output/include
LIBS+=$(BRPC_PATH)/output/lib

HDRPATHS = $(addprefix -I, $(HDRS))
LIBPATHS = $(addprefix -L, $(LIBS))
COMMA=,
SOPATHS=$(addprefix -Wl$(COMMA)-rpath=, $(LIBS))

STATIC_LINKINGS += -lthrift -lgflags -lbrpc -levent

CLIENT_SOURCES = client.cpp
CLIENT2_SOURCES = client2.cpp
SERVER_SOURCES = server.cpp
SERVER2_SOURCES = server2.cpp
PROTOS = $(wildcard *.proto)

PROTO_OBJS = $(PROTOS:.proto=.pb.o)
PROTO_GENS = $(PROTOS:.proto=.pb.h) $(PROTOS:.proto=.pb.cc)
CLIENT_OBJS = $(addsuffix .o, $(basename $(CLIENT_SOURCES)))
CLIENT2_OBJS = $(addsuffix .o, $(basename $(CLIENT2_SOURCES)))
SERVER_OBJS = $(addsuffix .o, $(basename $(SERVER_SOURCES)))
SERVER2_OBJS = $(addsuffix .o, $(basename $(SERVER2_SOURCES)))

.PHONY:all
all: echo_client echo_client2 echo_server echo_server2 native_server native_client

.PHONY:clean
clean:
	@echo "> Cleaning"
	rm -rf echo_client echo_client2 echo_server echo_server2 $(PROTO_GENS) $(PROTO_OBJS) $(CLIENT_OBJS) $(CLIENT2_OBJS) $(SERVER_OBJS) $(SERVER2_OBJS) native_server native_client EchoService.o echo_types.o libechothrift.a gen-cpp gen-py

echo_client:$(PROTO_OBJS) $(CLIENT_OBJS) libechothrift.a
	@echo "> Linking $@"
	$(CXX) $(LIBPATHS) $(SOPATHS) -Xlinker "-(" $^ -Xlinker "-)" $(STATIC_LINKINGS) $(DYNAMIC_LINKINGS) -o $@

echo_client2:$(PROTO_OBJS) $(CLIENT2_OBJS) libechothrift.a
	@echo "> Linking $@"
	$(CXX) $(LIBPATHS) $(SOPATHS) -Xlinker "-(" $^ -Xlinker "-)" $(STATIC_LINKINGS) $(DYNAMIC_LINKINGS) -o $@

echo_server:$(PROTO_OBJS) $(SERVER_OBJS) libechothrift.a
	@echo "> Linking $@"
	$(CXX) $(LIBPATHS) $(SOPATHS) -Xlinker "-(" $^ libechothrift.a -Xlinker "-)" $(STATIC_LINKINGS) $(DYNAMIC_LINKINGS) -o $@

echo_server2:$(PROTO_OBJS) $(SERVER2_OBJS) libechothrift.a
	@echo "> Linking $@"
	$(CXX) $(LIBPATHS) $(SOPATHS) -Xlinker "-(" $^ libechothrift.a -Xlinker "-)" $(STATIC_LINKINGS) $(DYNAMIC_LINKINGS) -o $@

%.o:%.cpp libechothrift.a
	@echo "> Compiling $@"
	$(CXX) -c $(HDRPATHS) $(CXXFLAGS) $< -o $@

%.o:%.cc libechothrift.a
	@echo "> Compiling $@"
	$(CXX) -c $(HDRPATHS) $(CXXFLAGS) $< -o $@

libechothrift.a:
	@echo "> Generating thrift files"
	thrift --gen cpp echo.thrift
	thrift --gen py echo.thrift
	$(CXX) -c gen-cpp/echo_types.cpp -o echo_types.o
	$(CXX) -c gen-cpp/EchoService.cpp -o EchoService.o
	ar -crv libechothrift.a EchoService.o echo_types.o

native_server: libechothrift.a
	$(CXX) native_server.cpp gen-cpp/echo_types.cpp gen-cpp/EchoService.cpp $(HDRPATHS) $(LIBPATHS) $(SOPATHS) $(CXXFLAGS) -lthriftnb -lthrift -levent -lpthread -lgflags -lbrpc -o native_server

native_client: libechothrift.a
	$(CXX) native_client.cpp gen-cpp/echo_types.cpp gen-cpp/EchoService.cpp $(HDRPATHS) $(LIBPATHS) $(SOPATHS) $(CXXFLAGS) -lthriftnb -lthrift -levent -lpthread -lgflags -lbrpc -o native_client
