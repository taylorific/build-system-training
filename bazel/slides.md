---
layout: section
---

# Bazel

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

# Bazelisk Install

```bash
apt-get update
apt-get install curl ca-certificates
```

```bash
# Intel
sudo curl -L \
  -o /usr/local/bin/bazelisk \
  https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64

sudo chmod +x /usr/local/bin/bazelisk
sudo ln -s /usr/local/bin/bazelisk /usr/local/bin/bazel
```

```bash
# ARM64
sudo curl -L \
  -o /usr/local/bin/bazelisk \
  https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-arm64

sudo chmod +x /usr/local/bin/bazelisk
sudo ln -s /usr/local/bin/bazelisk /usr/local/bin/bazel
```

---
hideInToc: true
---

```bash
$ bazelisk version
Bazelisk version: v1.28.1
WARNING: Invoking Bazel in batch mode since it is not invoked from within a workspace (below a directory having a MODULE.bazel file).
OpenJDK 64-Bit Server VM warning: Options -Xverify:none and -noverify were deprecated in JDK 13 and will likely be removed in a future release.
Build label: 9.0.0
Build target: @@//src/main/java/com/google/devtools/build/lib/bazel:BazelServer
Build time: Tue Jan 20 18:18:01 2026 (1768933081)
Build timestamp: 1768933081
Build timestamp as int: 1768933081

# ls ~/.cache/bazelisk
downloads
```

---
hideInToc: true
---

- https://bazel.build/install/completion#bash
- https://bazel.build/install/completion#zsh

---
hideInToc: true
---

```bash
# Intel
sudo curl -L -o /usr/local/bin/buildifier \
  https://github.com/bazelbuild/buildtools/releases/latest/download/buildifier-linux-amd64

sudo chmod +x /usr/local/bin/buildifier
```

```bash
# ARM64
sudo curl -L -o /usr/local/bin/buildifier \
  https://github.com/bazelbuild/buildtools/releases/latest/download/buildifier-linux-arm64

sudo chmod +x /usr/local/bin/buildifier
```

```bash
$ buildifier --version
buildifier version: 8.5.1-3-g0951e28
buildifier scm revision: 0951e28ad5a191e4d421d3899f3259e3e0f1aa54
```

---
hideInToc: true
---

```bash
apt-get update
apt-get install build-essential
DEBIAN_FRONTEND=noninteractive apt-get install vim
```

---
hideInToc: true
---

```bash
mkdir -p /workspace
cd /workspace

mkdir -p /workspace/example1

cat >example1/hello_world.cpp <<'EOF'
#include <iostream>

int main() {
  std::cout << "Hello, this is my first Bazel target" << std::endl;
  return 0;
}
EOF
```

---
hideInToc: true
---

```bash
touch MODULE.bazel

cat >example1/BUILD <<'EOF'
cc_binary(
    name = "hello_world",
    srcs = ["hello_world.cpp"],
)
EOF
```

---
hideInToc: true
---

```bash
$ bazel build //example1:hello_world
ERROR: Traceback (most recent call last):
	File "/workspace/example1/BUILD", line 1, column 10, in <toplevel>
		cc_binary(
	File "/virtual_builtins_bzl/bazel/exports.bzl", line 40, column 9, in _removed_rule_failure
Error in fail:
         This rule has been removed from Bazel. Please add a `load()` statement for it.
         This can also be done automatically by running:
         buildifier --lint=fix <path-to-BUILD-or-bzl-file>
```

```bash
$ buildifier -lint=fix -r .

$ cat example1/BUILD
load("@rules_cc//cc:cc_binary.bzl", "cc_binary")

cc_binary(
    name = "hello_world",
    srcs = ["hello_world.cpp"],
)
```

---
hideInToc: true
---

```bash
# No good way I know of to generate this
# Browse on https://registry.bazel.build/modules/rules_cc
$ cat >MODULE.bazel <<'EOF'
bazel_dep(name = "rules_cc", version = "0.2.17")
EOF

$ bazel build //example1:hello_world
```

---
hideInToc: true
---

```bash
bazel build //example1:hello_world
bazel run //example1:hello_world
bazel cquery --output=files //example1:hello_world
```

---
hideInToc: true
---

```bash
buildifier --mode=check --lint=off -v -r .

buildifier --mode=fix --lint=off -v -r .
```

```bash
buildifier --mode=fix --lint=warn -v -r .

buildifier --mode=fix --lint=fix -v -r .
```

---
hideInToc: true
---

```bash
cd /workspace
mkdir -p /workspace/example2

cat >example2/hello_library.h <<'EOF'
namespace hello_library {
    int sum( int a, int b );
}
EOF

cat >example2/hello_library.cpp <<'EOF'
#include "hello_library.h"

namespace hello_library {
    int sum( int a, int b )
    {
        return a + b;
    }
}
EOF
```

```bash
cat >MODULE.bazel <<'EOF'
bazel_dep(name = "rules_cc", version = "0.2.17")
EOF

cat >example2/BUILD <<'EOF'
load("@rules_cc//cc:defs.bzl", "cc_library")

cc_library(
    name = "hello_library",
    srcs = ["hello_library.cpp"],
    hdrs = ["hello_library.h"],
    visibility = ["//visibility:public"],
)
EOF
```

```bash
bazel build //example2:hello_library
```

---
hideInToc: true
---

```bash
cat >example2/hello_world.cpp <<'EOF'
#include "hello_library.h"
#include <iostream>

int main() {
  std::cout << "Hello world " << hello_library::sum(40, 2) << std::endl;
  return 0;
}
EOF
```

```bash
cat >example2/BUILD <<'EOF'
load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")

load("@rules_cc//cc:defs.bzl", "cc_library")

cc_library(
    name = "hello_library",
    srcs = ["hello_library.cpp"],
    hdrs = ["hello_library.h"],
    visibility = ["//visibility:public"],
)

cc_binary(
    name = "hello_world",
    srcs = ["hello_world.cpp"],
    deps = [":hello_library"],
)
EOF
```

```bash
bazel build //example2:hello_world
bazel run //example2:hello_world
```

---
hideInToc: true
---

```bash
cat >MODULE.bazel <<'EOF'
bazel_dep(name = "rules_cc", version = "0.2.17")
bazel_dep(name = "googletest", version = "1.17.0.bcr.2")
EOF
```

```bash
cat >example2/BUILD <<'EOF'
load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_test.bzl", "cc_test")
load("@rules_cc//cc:defs.bzl", "cc_library")

cc_library(
    name = "hello_library",
    srcs = ["hello_library.cpp"],
    hdrs = ["hello_library.h"],
    visibility = ["//visibility:public"],
)

cc_binary(
    name = "hello_world",
    srcs = ["hello_world.cpp"],
    deps = [":hello_library"],
)

cc_test(
    name = "hello_test",
    srcs = ["hello_test.cpp"],
    deps = [
        ":hello_library",
        "@googletest//:gtest_main",
    ],
)
EOF
```

```bash
cat >example2/hello_test.cpp <<'EOF'
#include "hello_library.h"
#include <gtest/gtest.h>

TEST(Sum, SumbNegativeValues) {
    EXPECT_EQ( -11, hello_library::sum(-8, -3) );
}

TEST(Sum, SumPositiveValues) {
    EXPECT_EQ( 4, hello_library::sum(2, 2) );
}
EOF
```

```bash
bazel build //example2:hello_test
bazel test //example2:hello_test
```

---
hideInToc: true
---

```bash
bazel cquery "deps(//example2:hello_world)"

bazel cquery \
  'filter("^//", deps(//:hello_world))' \
  --output=label

bazel cquery "allpaths(//:hello_world,//:hello_library.h)"

bazel cquery "somepath(//:hello_world,//:hello_library.h)"

bazel cquery "rdeps(...,//:hello_library.h)"
```

---
layout: section
---

## Rules

<br>
<br>
<Link to="toc" title="Table of Contents"/>
