---
layout: section
---

# Jujutsu

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

# What is Jujutsu?

Jujutsu (`jj`) is a version control system.

It combines ideas from:

- Git
- Mercurial
- Darcs / Pijul
- Google's internal development workflows

while remaining compatible with Git repositories.

---
hideInToc: true
---

# Installing Jujutsu - Linux

Download the latest release from GitHub:

```bash
VERSION=$(
  curl -fsSL https://api.github.com/repos/jj-vcs/jj/releases/latest |
  sed -n 's/.*"tag_name": *"v\([^"]*\)".*/\1/p'
)

ARCH=$(uname -m)

curl -fLO \
  "https://github.com/jj-vcs/jj/releases/download/v${VERSION}/jj-v${VERSION}-${ARCH}-unknown-linux-musl.tar.gz"

# Extract and install
tar -xzf "jj-v${VERSION}-${ARCH}-unknown-linux-musl.tar.gz"

sudo install jj /usr/local/bin/jj

# Verify
jj --version
```

---
hideInToc: true
---

# Installing Jujutsu - macOS

Download the latest release from GitHub:

```bash
Download the latest release directly from GitHub:

```bash
VERSION=$(
  curl -fsSL https://api.github.com/repos/jj-vcs/jj/releases/latest |
  sed -n 's/.*"tag_name": *"v\([^"]*\)".*/\1/p'
)

ARCH=$(uname -m)

curl -fLO \
  "https://github.com/jj-vcs/jj/releases/download/v${VERSION}/jj-v${VERSION}-${ARCH}-apple-darwin.tar.gz"

# Extract and install
tar -xzf "jj-v${VERSION}-${ARCH}-apple-darwin.tar.gz"

sudo install jj /usr/local/bin/jj

# Verify
jj --version
```

---
hideInToc: true
---

# Installing Jujutsu - Windows

Install the latest release with WinGet:

```powershell
winget install jj-vcs.jj
```

---
hideInToc: true
layout: two-cols
---

# References

**Martin von Zweigbergk**  
*Jujutsu: A Git-Compatible VCS*  
Git Merge 2022  
https://github.com/jj-vcs/jj/wiki/Media

**Martin von Zweigbergk**  
*Jujutsu: A Git-compatible VCS*  
Git Merge 2024  
https://github.com/jj-vcs/jj/wiki/Media

**Jujutsu Project**  
*Jujutsu Documentation*  
https://jj-vcs.github.io/jj/latest/

**Jujutsu Project**  
*Jujutsu Source Repository*  
https://github.com/jj-vcs/jj

::right::

# &nbsp;

**Rachel Potvin and Josh Levenberg**  
*Why Google Stores Billions of Lines of Code in a Single Repository*  
Communications of the ACM, Vol. 59, No. 7, 2016  
[DOI: 10.1145/2854146](https://doi.org/10.1145/2854146)

**Andrey Mokhov, Neil Mitchell, and Simon Peyton Jones**  
*Build Systems à la Carte*  
Proceedings of the ACM on Programming Languages, Vol. 2, ICFP, 2018  
[DOI: 10.1145/3236774](https://doi.org/10.1145/3236774)
