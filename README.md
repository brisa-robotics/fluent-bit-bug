# flb_config_map_set crash with fluent bit when compiling c plugin

To reproduce:

On host:

```bash
$ git clone git@github.com:brisa-robotics/fluent-bit-bug.git --submodules
$ docker run \
    --name fluent-bit-external-input-c-plugin \
    -v ${PWD}:/ws \
    --workdir=/ws \
    -it \
    ubuntu:jammy
```

In the container
```bash
$ apt update -qq
$ apt install -y cmake make g++ vim tree pkg-config libyaml-dev flex bison git libssl-dev
$ cd fluent-bit/build
$ cmake \
    -DBUILD_TESTING=Off \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DFLB_EXAMPLES=Off \
    -DFLB_SHARED_LIB=On \
    -DFLB_PROXY_GO=On ..
$ make -j
# Workaround to bypass the include that can't be found.
# This can be fixed later more permanently by adding it in the include/CMakeLists.txt in the fluent-bit repo.
$ cp -r /ws/fluent-bit/build/lib/monkey/include/monkey/mk_core/ /ws/fluent-bit/include/
```

Then let's compile our custom plugin
```bash
cd /ws/fluent-bit-plugin
mkdir build
cd build
cmake -DFLB_SOURCE=/ws/fluent-bit -DPLUGIN_NAME=in_dummy2 ../
make
```

And finally, running it:
```bash
/ws/fluent-bit/build/bin/fluent-bit -e ./flb-in_dummy2.so -i dummy2 -o stdout
```

Leading to the same error 🚀
```
Fluent Bit v2.0.7
* Copyright (C) 2015-2022 The Fluent Bit Authors
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2022/12/08 13:39:07] [ info] [fluent bit] version=2.0.7, commit=fa7fd13c1f, pid=23255
[2022/12/08 13:39:07] [ info] [storage] ver=1.3.0, type=memory, sync=normal, checksum=off, max_chunks_up=128
[2022/12/08 13:39:07] [ info] [cmetrics] version=0.5.7
[2022/12/08 13:39:07] [ info] [ctraces ] version=0.2.5
[2022/12/08 13:39:07] [ info] [input:dummy2:dummy2.0] initializing
[2022/12/08 13:39:07] [engine] caught signal (SIGSEGV)
[2022/12/08 13:39:07] [ info] [input:dummy2:dummy2.0] storage_strategy='memory' (memory only)
#0  0x5604725d22e0      in  flb_config_map_set() at src/flb_config_map.c:591
#1  0x7fbeb82f21b8      in  ???() at ???:0
#2  0x7fbeb82f2740      in  ???() at ???:0
#3  0x7fbeb82f2b1e      in  ???() at ???:0
#4  0x5604725b00f3      in  flb_input_instance_init() at src/flb_input.c:1162
#5  0x5604725b029c      in  flb_input_init_all() at src/flb_input.c:1216
#6  0x5604725e6b13      in  flb_engine_start() at src/flb_engine.c:717
#7  0x560472589224      in  flb_lib_worker() at src/flb_lib.c:629
#8  0x7fbeb8390b42      in  ???() at ???:0
#9  0x7fbeb8421bb3      in  ???() at ???:0
#10 0xffffffffffffffff  in  ???() at ???:0
Aborted (core dumped)
```
