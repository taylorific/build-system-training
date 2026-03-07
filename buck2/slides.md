---
layout: section
---

# BUCK2

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

## Bazelisk Install

```bash
sudo apt-get update
sudo apt-get install curl ca-certificates zstd
```

```bash
# Intel
curl -L  https://github.com/facebook/buck2/releases/download/latest/buck2-x86_64-unknown-linux-musl.zst \
  | zstd -d \
  | tee /usr/local/bin/buck2 > /dev/null

sudo chmod +x /usr/local/bin/buck2
```

```bash
# ARM64
curl -L https://github.com/facebook/buck2/releases/download/latest/buck2-aarch64-unknown-linux-musl.zst \
  | zstd -d \
  | tee /usr/local/bin/buck2 > /dev/null

chmod +x /usr/local/bin/buck2
```

---
hideInToc: true
---

BUCK2 is similar to Bazel but:
- configuration lives in `.buckconfig`
- rules live in `BUCK` files
- commands are `buck2 build` and `buck2 run`

---
hideInToc: true
---

```bash
apt-get update
apt-get install git build-essential
```

---
hideInToc: true
---

```bash
mkdir -p /workspace
cd /workspace
buck2 init hello

cd /workspace/hello
```

```bash
buck2 build //:hello_world --show-output
```

```bash
# List targets
buck2 query //:

# Show dependency graph
buck2 query "deps(//:hello_world)"

# Clean builds
buck2 clean
```
