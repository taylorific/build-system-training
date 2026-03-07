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

## Bazelisk Install

```bash
sudo apt-get update
sudo apt-get install curl ca-certificates
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

## Bazelisk Install

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

## Install C++ toolchain

```bash
sudo apt-get update
sudo apt-get install build-essential
sudo -E DEBIAN_FRONTEND=noninteractive apt-get install vim
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

---
hideInToc: true
---

```bash
mkdir -p /workspace/rules
touch /workspace/rules/MODULE.bazel

mkdir -p /workspace/rules/hello
cat >/workspace/rules/hello/hello.bzl <<'EOF'
def _hello_impl(ctx):
    content = ctx.attr.content

    # Declare output file
    output = ctx.actions.declare_file(ctx.label.name + ".txt")

    # Write the file
    ctx.actions.write(
        output = output,
        content = content,
    )

    # Tell Bazel what files this rule produces
    return [DefaultInfo(files = depset([output]))]

hello_rule = rule(
    implementation = _hello_impl,
    attrs = {
        "content": attr.string(),
    },
)
EOF
```

---
hideInToc: true
---

```bash
cat >/workspace/rules/hello/BUILD <<'EOF'
load("//hello:hello.bzl", "hello_rule")

hello_rule(
  name = "hello",
  content = "Hello Bazel\n",
)
EOF
```

```bash
cd /workspace/rules
bazel build //hello:hello

cat bazel-bin/hello/hello.txt
```

---
hideInToc: true
---

```bash
mkdir -p /workspace/rules/genrule
cat >/workspace/rules/genrule/BUILD <<'EOF'
genrule(
    name = "hello",
    outs = ["hello.txt"],
    cmd = "echo 'Hello from Bazel' > $@",
)
EOF

bazel build //genrule:hello
```

---
layout: section
---

## Python

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

## Python Toolchain

```bash
apt-get update
apt-get install -y python3
curl -LsSf https://astral.sh/uv/install.sh | sh
```

```bash
mkdir -p /workspace/python/app
cat >/workspace/python/app/.python-version <<'EOF'
3.12
EOF

cat >/workspace/python/app/pyproject.toml <<'EOF'
[project]
name = "app"
version = "0.1.0"
requires-python = ">=3.12,<3.13"
dependencies = [
  "requests>=2.32,<3",
]
EOF

cd /workspace/python/app
export PATH="/root/.local/bin:$PATH"
uv python install
uv lock
uv pip compile pyproject.toml -o requirements.txt
```

---
hideInToc: true
---

```bash
cat >/workspace/python/app/lib.py <<'EOF'
import requests

SYSTEM_CA_BUNDLE = "/etc/ssl/certs/ca-certificates.crt"

def fetch_status(url: str) -> int:
    response = requests.get(url, timeout=5, verify=SYSTEM_CA_BUNDLE)
    response.raise_for_status()
    return response.status_code
EOF

cat >/workspace/python/app/main.py <<'EOF'
from lib import fetch_status

def main() -> None:
    code = fetch_status("https://example.com")
    print(f"status={code}")

if __name__ == "__main__":
    main()
EOF

mkdir -p /workspace/python/app/tests
cat >/workspace/python/app/tests/lib_test.py <<'EOF'
from lib import fetch_status

def test_smoke():
    # For a real test, you'd usually mock requests instead of doing network I/O.
    assert callable(fetch_status)
EOF
```

---
hideInToc: true
---

```bash
uv sync
uv run python main.py
```

---
hideInToc: true
---

```bash
cat >/workspace/python/app/.bazelrc << 'EOF'
build --@rules_python//python/config_settings:bootstrap_impl=script
run --@rules_python//python/config_settings:bootstrap_impl=script
test --@rules_python//python/config_settings:bootstrap_impl=script
EOF
```

```bash
cat >/workspace/python/app/MODULE.bazel <<'EOF'
bazel_dep(name = "rules_python", version = "1.7.0")

python = use_extension(
    "@rules_python//python/extensions:python.bzl",
    "python",
)

python.defaults(
    python_version = "3.12",
)

python.toolchain(
    python_version = "3.12",
)

pip = use_extension(
    "@rules_python//python/extensions:pip.bzl",
    "pip",
)

pip.parse(
    hub_name = "pypi",
    python_version = "3.12",
    requirements_lock = "//:requirements.txt",
)

use_repo(python, "python_3_12")
use_repo(pip, "pypi")
EOF
```

```bash
cat >/workspace/python/app/BUILD.bazel <<'EOF'
load("@rules_python//python:defs.bzl", "py_binary", "py_library", "py_test")

py_library(
    name = "lib",
    srcs = ["lib.py"],
    deps = [
        "@pypi//requests:pkg",
    ],
)

py_binary(
    name = "app",
    srcs = ["main.py"],
    main = "main.py",
    deps = [":lib"],
)

py_test(
    name = "lib_test",
    srcs = ["tests/lib_test.py"],
    deps = [":lib"],
)
EOF
```

---
hideInToc: true
---

```bash
# Use uv for local Python workflow
uv add rich
uv lock
uv pip compile pyproject.toml -o requirements.txt

# Use Bazel for build/text
bazel run //:app
bazel test //:lib_test
```

---
layout: section
---

## Typescript

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

## Typescript Toolchain

```bash
apt-get update
apt-get install -y --no-install-recommends \
  curl \
  ca-certificates \
  git \
  python3 \
  build-essential \
  xz-utils

export NODE_VERSION=20.11.1
export PNPM_VERSION=10.17.1

curl -fsSL https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-arm64.tar.xz \
  | tar -xJ -C /usr/local --strip-components=1

corepack enable
corepack prepare pnpm@${PNPM_VERSION} --activate

node -v
pnpm -v
```

---
hideInToc: true
---

```bash
mkdir -p /workspace/typescript/app/
cat >/workspace/typescript/app/package.json <<'EOF'
{
  "name": "ts_app",
  "private": true,
  "type": "module",
  "devDependencies": {
    "typescript": "5.8.2"
  }
}
EOF

cat >/workspace/typescript/app/tsconfig.json <<'EOF'
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "moduleResolution": "node",
    "rootDir": "src",
    "strict": true
  }
}
EOF

cd /workspace/typescript/app
pnpm install
```

```bash
cat >/workspace/typescript/app/MODULE.bazel <<'EOF'
bazel_dep(name = "aspect_rules_js", version = "3.0.2")
bazel_dep(name = "aspect_rules_ts", version = "3.8.6")

npm = use_extension("@aspect_rules_js//npm:extensions.bzl", "npm")

npm.translate_lock(
    name = "npm",
    pnpm_lock = "//:pnpm-lock.yaml",
)

use_repo(npm, "npm")

rules_ts_ext = use_extension(
    "@aspect_rules_ts//ts:extensions.bzl",
    "ext",
    dev_dependency = True,
)

rules_ts_ext.deps(
    ts_version_from = "//:package.json",
)

use_repo(rules_ts_ext, "npm_typescript")
EOF

cat >/workspace/typescript/app/BUILD.bazel <<'EOF'
load("@aspect_rules_ts//ts:defs.bzl", "ts_project", "ts_config")
load("@aspect_rules_js//js:defs.bzl", "js_binary")

ts_config(
    name = "tsconfig",
    src = "tsconfig.json",
)

ts_project(
    name = "app_lib",
    srcs = [
        "src/main.ts",
        "src/lib.ts",
    ],
    tsconfig = ":tsconfig",
    transpiler = "tsc",
)

js_binary(
    name = "app",
    entry_point = "src/main.js",
    data = [":app_lib"],
)
EOF
```

```bash
mkdir -p /workspace/typescript/app/src
cat >/workspace/typescript/app/src/lib.ts <<'EOF'
export function formatStatus(code: number): string {
  return `status=${code}`;
}

cat >/workspace/typescript/app/src/main.ts <<'EOF'
import { formatStatus } from "./lib.js";

console.log(formatStatus(200));
EOF
```

---
hideInToc: true
---

```bash
bazel build //:app
bazel run //:app
```

---
layout: section
---

## Go

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

## Go Toolchain

```bash
sudo apt-get update
sudo apt-get install -y --no-install-recommends \
  curl ca-certificates

export GO_VERSION=1.24.0
curl -fsSLO https://go.dev/dl/go${GO_VERSION}.linux-arm64.tar.gz \
  && rm -rf /usr/local/go \
  && tar -C /usr/local -xzf go${GO_VERSION}.linux-arm64.tar.gz \
  && rm go${GO_VERSION}.linux-arm64.tar.gz

export PATH="/usr/local/go/bin:${PATH}"

go version
```

---
hideInToc: true
---

```bash
mkdir -p /workspace/go/app/
cat >/workspace/go/app/go.mod <<'EOF'
module example.com/go_app

go 1.24

require github.com/google/uuid v1.6.0
EOF

cat >/workspace/go/app/MODULE.bazel <<'EOF'
bazel_dep(name = "rules_go", version = "0.60.0")
bazel_dep(name = "gazelle", version = "0.47.0")

go_sdk = use_extension("@rules_go//go:extensions.bzl", "go_sdk")
go_sdk.download(version = "1.24.6")

go_deps = use_extension("@gazelle//:extensions.bzl", "go_deps")
go_deps.from_file(go_mod = "//:go.mod")

use_repo(go_deps, "com_github_google_uuid")
EOF

cat >/workspace/go/app/BUILD.bazel <<'EOF'
load("@rules_go//go:def.bzl", "go_binary")
load("@gazelle//:def.bzl", "gazelle")

# gazelle:prefix example.com/go_app

gazelle(
    name = "gazelle",
)

go_binary(
    name = "app",
    srcs = ["main.go"],
    importpath = "example.com/go_app",
    deps = ["//lib:lib"],
)
EOF

cat >/workspace/go/app/lib/BUILD.bazel <<'EOF'
load("@rules_go//go:def.bzl", "go_library", "go_test")

go_library(
    name = "lib",
    srcs = ["lib.go"],
    importpath = "example.com/go_app/lib",
    visibility = ["//visibility:public"],
    deps = ["@com_github_google_uuid//:uuid"],
)

go_test(
    name = "lib_test",
    srcs = ["lib_test.go"],
    embed = [":lib"],
)
EOF
```

```bash
mkdir -p /workspace/go/app/lib/
cat >/workspace/go/app/lib/lib.go <<'EOF'
package lib

import (
	"fmt"

	"github.com/google/uuid"
)

func FormatStatus(code int) string {
	return fmt.Sprintf("status=%d id=%s", code, uuid.Nil.String())
}
EOF

cat >/workspace/go/app/lib/lib_test.go <<'EOF'
package lib

import (
	"strings"
	"testing"
)

func TestFormatStatus(t *testing.T) {
	got := FormatStatus(200)

	if !strings.Contains(got, "status=200") {
		t.Fatalf("expected status=200 in %q", got)
	}

	if !strings.Contains(got, "00000000-0000-0000-0000-000000000000") {
		t.Fatalf("expected nil UUID in %q", got)
	}
}
EOF

cat >/workspace/go/app/main.go <<'EOF'
package main

import (
	"fmt"

	"example.com/go_app/lib"
)

func main() {
	fmt.Println(lib.FormatStatus(200))
}
EOF
```

---
hideInToc: true
---

```bash
bazel build //:app
bazel test //lib:lib_test
bazel run //:app
```
