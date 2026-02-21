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
mkdir -p /workspace
cd /workspace

cat >MODULE.bazel <<'EOF'
bazel_dep(name = "rules_cc", version = "0.2.14")
EOF

cat >BUILD <<'EOF'
load("@rules_cc//cc:cc_binary.bzl", "cc_binary")

cc_binary(
    name = "hello_world",
    srcs = ["hello_world.cpp"],
)
EOF

cat >hello_world.cpp <<'EOF'
#include <iostream>

int main() {
  std::cout << "Hello, this is my first Bazel target" << std::endl;;
  return 0;
}
EOF
```

```bash
bazel build //:hello_world
bazel run //:hello_world
bazel cquery --output=files //:hello_world
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
mkdir -p /workspace
cd /workspace

cat >MODULE.bazel <<'EOF'
bazel_dep(name = "rules_cc", version = "0.2.14")
EOF

cat >BUILD <<'EOF'
load("@rules_cc//cc:defs.bzl", "cc_library")

cc_library(
    name = "hello_library",
    srcs = ["hello_library.cpp"],
    hdrs = ["hello_library.h"],
    visibility = ["//visibility:public"],
)
EOF

cat >hello_library.h <<'EOF'
namespace hello_library {
    int sum( int a, int b );
}
EOF

cat >hello_library.cpp <<'EOF'
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
bazel build //:hello_library
```

---
hideInToc: true
---

```bash
cat >BUILD <<'EOF'
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
cat >hello_world.cpp <<'EOF'
#include "hello_library.h"
#include <iostream>

int main() {
  std::cout << "Hello world " << hello_library::sum(40, 2) << std::endl;
  return 0;
}
EOF
```

```bash
bazel build //:hello_world
bazel run //:hello_world
```
