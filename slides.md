---
theme: seriph
background: https://i.ibb.co/khMnPgg/killarney.jpg
title: Teach Old Code New Tricks
info: Dor Schwartz @ PyCon IL 2024
drawings:
  persist: true
transition: fade
hideInToc: true
layout: cover
colorSchema: light
zoom: 0.9
---
  
# Teach Old Code New Tricks
## Automating Code Quality in Large Projects

<br>
<br>
Dor Schwartz

_Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)_

---
title: Agenda
hideInToc: true
---

# Agenda 
<br>
<Toc maxDepth="1"></Toc>

<!-- 
- Hard to get into others' code, sometimes our own
- The more devs, time, code, the harder to write uniformly, potential for issues rise
- Detect errors closer to the dev, shift left
- Especially in Python, self-taught friendly - easy to write, hard to maintain
 -->

---
title: What is code quality?
---
# Quality code is... 

<v-clicks>

- Clear
- Persistent
- Performant
- Safe
- Secure
- Documented?
- Tested?

</v-clicks>

<!-- 
Python makes it easy to slip up

- *Clear*: coherent, easy to understand each part
- *Persistant*: easier to read, less conflicts
- *Performant*: trivial, some traps in Python
- *Safe*: bare except/Excecption may hide issues
- *Secure*: See Ran's talk

Last two - won't be discussed today.
 -->

---
level: 2
---
# Lack of quality becomes debt

<v-clicks>

- Complex/inconsistent code is harder to debug 🪲
- Decreased reusability 🗑️
- Performance issues 🐢
- Impaired collaboration 💔
- Longer onboarding 😒

</v-clicks>
<!-- 
Tech debt grows with time&code.
- Suddenly old code we had no time for typing, crashes because of a surprise `None`
- Complex functions, multiple uses/inputs/outputs are too specific and hard to debug
- Collaboration - untyped/complex code, only some know what's up
 -->
---
level: 2
---

# Types of Code Quality Tools

<v-clicks>

- Linters
- Formatters
- Type checkers
- Refactoring as a Service

</v-clicks>

<!-- 
The community built tools to prevent pitfalls. Most OSS, most Python.

We'll discuss categories, tools and methods for applying

Tools may be opinionated. They have the last word. 
 -->

---
level: 2
---
# BYOLinter

<v-clicks>

- You make the rules
- Automatically **mod**ify your **cod**e with **codemod**s (using `LibCST`)
  - See [Soof Golan's 20m video](https://www.youtube.com/watch?v=mINwGOGLRhg)
- Yishai Zinkin's [Unlocking Python's AST](https://cfp.pycon.org.il/pycon-2024/talk/SLGDWK/) talk @ **14:00** in this hall
</v-clicks>

<!-- 

Enforce custom standards

 -->
---
level: 2
---
# Advantages of quality code
<br>
<v-clicks>

- Allow devs to focus on **coding** 
- First line of defense
- Easier on reviewers
- ~~Less~~ no style arguments
- The _pedantic senior on your shoulder_
- Way to learn about more features & edge cases of Python

</v-clicks>

<!-- 
the freedom to not care about formatting, import order
"The academy"
 -->

---
level: 2
---

# Git's `pre-commit` hook

`.git/hooks/pre-commit`
<v-click>
```bash
# If any command fails, exit with same code
set -eo pipefail

ruff check src
echo "ruff passed!"

mypy src --check
echo "mypy passed!"
```

Runs on all files - could take a *lot* of time in large projects
</v-click>

<!-- 
Git has a builtin ability to run scripts before committing
A lot of time - we don't want to wait 2m before committing
 -->
---
level: 2
title: The `pre-commit` framework
---

# The Pre-commit Framework

- Runs a list of hooks **consistently**
- Configuration file in the repository 
  - Single source of configurations and tool versions
  - Versioned
- Creates an isolated virtual environment for each hook

<!-- 
- Single source - make sure everyone adheres to the same rules (no version/rule diff)
- Tools can be in many languages
- Runs in three places: on command, before commit (optional), and in CI
 -->
---
level: 3
---

### Configuring `pre-commit` hooks
`.pre-commit-config.yaml`

````md magic-move
```yaml{1|2|2-3|4-6|7-9}
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.4.0
  hooks:
  - id: check-yaml
    exclude: .vscode/.*
  - id: trailing-whitespace
    args:
    - --markdown-linebreak-ext=md
```
````
---
level: 3
---

### Creating `pre-commit` hooks

`.pre-commit-hooks.yaml`
````md magic-move
```yaml{1-2|3|4-5|6|7-9}
- id: format
  name: formatter
  description: Format non-code files, removing redundant fields
  entry: demisto-sdk
  args: ["format", "--no-validate", "-g"]
  language: python
  pass_filenames: false
  require_serial: true
```
````

---
level: 1
---

# Configuring code quality tools
<br>
Choose to check, or ignore
<v-clicks>

- Specified rules
- Rules by severity/category
- Specific files/pattern
- Specific rules for file/pattern

Other settings
- When to autofix
- Rule arguments
- Much more
</v-clicks>
---
level: 2
---

# Pyproject.toml

````md magic-move
```toml
[project]
name = "example-package"
version = "0.1.0"
description = "A small example package"
authors = [{ name = "Dev", email = "Dev@example.com" }]
dependencies = ["requests>=2.25.1"]
requires-python = ">=3.8"
```

```toml {*|1|1-2|4-7|9-10}
[tool.ruff]
line-length = 130

[tool.ruff.lint]
select = ["E", "F", "I"]
ignore = ["E501"]
target-version = "py312"

[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]
```
````

---
title: Challenges introducing new tools
---

# So you want to introduce a new tool
<v-clicks depth=2>

- Developers start getting headaches 🤕
  - Old code fails new rules
  - Merge conflicts 
  - PRs with fixes grow beyond their intended size

- Git mess
  - `git diff` is harder to review
  - `git blame` whose line was it anyway? 🤔

</v-clicks>

<!-- 
- You are convinced, these are awesome tools
- You can keep running barefoot 
-->

---
level: 2
---

# Adding tools quickly, and very dirtily 

<v-clicks depth=2>

- **Bang your head against the wall**: Require developers to fix all errors manually
  - Not fun
  - Distracts developers
  - Slows down development
  - Messes up `git diff` & `blame`

- **Bury your head in the sand**: Ignore pre-existing violations
  - Easy
  - Sweeping mess under the mat != cleaning
</v-clicks>

<!--
Like every job interview, let's discuss the naive solution
-->

---
level: 2
---

## Our challange
<v-clicks every=2>

**Team** 👩‍💻
  - ~50 developers working on the repo
  - Open source, customer & partner contributions

**Repo** 🏗️
  - ~1,175,000 lines of code
  - 8 years of repo history

**Structure** 🧩

  - Monorepo of (almost) independent, lambda-like scripts
  - Each script uses a different Python version & dependencies
</v-clicks>

<!-- 
- large company, need to keep standards
- Can't just apply a linter to the whole repo, we wrapped pre-commit and generate it dynamically
 -->

---
title: Teaching old code new tricks™
---

# Let’s teach old code new tricks  🎓
Boy, there are lots of tricks

<v-clicks depth=2>

- **The No Brainer** 
  - Apply categories / trust the defaults
  - *Cost*: no control of new rules or categories

- **The Pedantic** 
  - Manually select rules
  - Use data from your repo
  - Form a committee: developers, leads, architects
  - *Cost*: time, effort

</v-clicks>
<!-- 
- Things can be easy, at a cost
- Scale of effort
 -->
---
level: 3
---
# Let your data guide you
##
What’s already adhered / violated in the project? 🕸️
<v-clicks>

- Using the code quality tools
- Use external tools *(shameless self-promotion)*

</v-clicks>

---
level: 3
title: adopt-ruff example
zoom: 0.7
---
<br>
<br>
<br>
<br>
```md
### adopt-ruff report for ScDor/my-dummy-repo (ruff 0.6.5)
#### Respected Ruff rules
374 Ruff rules are already respected in the repo - they can be added right away 🚀

| Code     | Name                                 | Fixable | Preview | Linter          |
| -------- | ------------------------------------ | ------- | ------- | --------------- |
| A003     | builtin-attribute-shadowing          | No      | False   | flake8-builtins |
| ASYNC100 | blocking-http-call-in-async-function | No      | False   | flake8-async    |
| PERF403  | manual-dict-comprehension            | No      | True    | Perflint        |
```

---
level: 2
title: Introduction Strategies
---

# Strategies
##
<v-click>
🧑‍⚖️ Rule-incremental
</v-click>
<br>

<v-click>

📂 Path-incremental
</v-click>

<v-click>
📑 Diff-based
</v-click>

---
level: 2
title: Rule-incremental
---

# Rule-incremental adoption

### Start small & grow gradually
<v-clicks>

- Choose a few rules ✔️
- Developers get used to the tools & see their value 💪
- Gradually add rules 📈
- *Extra mile:* Gamify fixing old pre-existing violations 🏆
</v-clicks>

<!-- Dependes on the project structure, usually possible -->
---
title: Rule-incremental approaches
level: 3
---

# Rule-incremental adoption

### Approaches
<v-clicks>

- Most important rules first ☝️
- Least violated rules first 📊
- `Error`s first 🚨
</v-clicks>

<!-- 
We decided on a threshold of violations, under which we enforce rules on old code 
 -->
---
title: Multiple configurations
level: 3
---

# Rule-incremental adoption

### Multiple configurations

- **Strict**: used in pre-commit & CI
- **Broad** only in IDEs: more rules

<!-- 
- Come with a cost 
- Can also just use a different config in CI
-->
---
level: 2
---
# Path-incremental adoption
<v-clicks>

- Compile a list of paths to check _(or to ignore)_ ✍️
- Always check new files ✨
- Gradually increase the number of files being checked 📈
- Open automated PRs with fixes 🦾

</v-clicks>


---
level: 3
title: Git blame ignore-revs
---
# Path-incremental adoption
`.git-blame-ignore-revs`
<v-click>
```
# changed line-length from 79 to 120 
11bcb1692a4344c08dc417101355fa42de61969b

# added pycln, updated pyupgrade, set --py38-plus
86095f6216a34547f94a695e1c0b1decaa9b339e
```
</v-click>
<br>
<v-click>
Developers will have to run a one-liner to use locally
```bash
git config blame.ignoreRevsFile .git-blame-ignore-revs
```
</v-click>

<br>


---
level: 2
---

# Diff-based adoption
Filter violations, only fail new ones
<v-clicks depth=2>

- Tools with a built-in `diff` feature
- meta-linters / add-ons / wrappers 
  - [**ondivi**](https://github.com/blablatdinov/ondivi) *only-diff-violations*
  - [**mypy-baseline**](https://github.com/orsinium-labs/mypy-baseline)
  - [**riff**](https://github.com/dorschw/riff) *ruff-diff*
</v-clicks>

---
level: 3
---
# mypy-baseline example
```
Your changes introduced new violations.
Please, resolve the violations above before moving forward.
Mypy is your friend.
```
```
total errors:
  fixed: 1
  new: 5
  unresolved: 480

errors by error code:
  arg-type      44   +1
  union-attr    30   -3
```

---
---
# Developer experience & recap 
- Single source of configurations & tool versions
- Communicating tool meaning, impact and importance
- Common errors and how to fix them
- Add linters without causing *tool fatigue*
- Ignoring errors only as last resort

<!-- 
careful - with great power 
misconfigured linters may frustrate devs, making them solve the symptom rather than the issue
-->
---
---

# Tool recommendations

<v-click>

## Linting
- [`ruff`](https://docs.astral.sh/ruff/) is blazing fast, replaces *flake8* and many of its plugins
- ... yet [`pylint`](https://www.pylint.org/) still pulls some unique tricks.

## Type Checking
- [`mypy`](https://mypy-lang.org/) is the common solution
- [`pyre`](https://pyre-check.org/) and [`pyright`](https://github.com/microsoft/pyright) are faster, with slight differences

<v-click>

*Rumor has it that `ruff` may [join](https://github.com/astral-sh/ruff/issues?q=sort%3Aupdated-desc+is%3Aissue+is%3Aopen+label%3Ared-knot) this party too 👀*
</v-click>
</v-click>

---
hideInToc: ture

---

# Tool recommendations
## Formatting
- `ruff-format` is super fast, 99.9%-compatible `black` replacement
#### Import sorting
- `ruff`'s `I` category replaces `isort` for import sorting

## Refactoring as a service
[`Sourcery`](https://sourcery.ai/) - free for OSS, does cool stuff

---
hideInToc: ture
---
# Resources 
- [A tale of two kitchens](https://dev.to/ldrscke/a-tale-of-two-kitchens-hypermodernizing-your-python-code-base-3jnh) / Christian Ledermann
- [Python Linters at scale](https://www.youtube.com/watch?v=-ihC3frDxss) / Jimmy Lai
- [Type annotation via automated refactoring](https://medium.com/building-carta/type-annotation-via-automated-refactoring-fd8edfe123d4) / Jimmy Lai
- [How to use git pre-commit hooks, the hard way and the easy way](https://verdantfox.com/blog/how-to-use-git-pre-commit-hooks-the-hard-way-and-the-easy-way) / Teddy Williams
- [Static Analysis at Scale: An Instagram Story](https://instagram-engineering.com/static-analysis-at-scale-an-instagram-story-8f498ab71a0c) / Benjamin Woodruff
- [pre-commit vs CI](https://switowski.com/blog/pre-commit-vs-ci/) / Sebastian Witowski
- [Linters aren't in your way. They're on your side](https://stackoverflow.blog/2020/07/20/linters-arent-in-your-way-theyre-on-your-side/) / Medi Madelen Gwosdz
- XKCD: [Code Quality 1](https://xkcd.com/1513/), [2](https://xkcd.com/1695/), [3](https://xkcd.com/1833/)
- [Writing Codemods](https://www.youtube.com/watch?v=mINwGOGLRhg) / Soof Golan
---
layout: cover
background: https://i.ibb.co/khMnPgg/killarney.jpg
hideInToc: ture
---
# Q&A 

<br>

## [/in/dor-schwartz](https://www.linkedin.com/in/dor-schwartz/)

---
hideInToc: ture
layout: cover
background: https://i.ibb.co/khMnPgg/killarney.jpg
---
# Some Examples

---
level: 3
title: Example - Formatting
---

# Formatting

````md magic-move
```python
def messy_function(a,b ,  c  ):
    if a>b:
     return \
 c
    return a+b

class MessyClass(NamedTuple):
  name: str
  def greet(self):
    print(f'Hello, {self.name}!'  )

```

```python
def not_as_messy_function(a, b, c):
    if a > b:
        return c
    return a + b


class MessyClass(NamedTuple):
    name: str

    def greet(self):
        print(f"Hello, {self.name}!")

```
````


---
level: 3
title: Example - Dependency Sorting
---

# Dependency Sorting 

````md magic-move
```python
from my_lib import Obj
import os

from my_lib import Obj2

from third_party import lib2, lib1
import os
from __future__ import absolute_import
from third_party import lib3
```

```python
from __future__ import absolute_import

import os

from third_party import (lib1, lib2, lib3)

from my_lib import Obj, Obj2
```
````
---
level: 3
title: Example - Type Checking
zoom: 0.85
---
 
# Type Checking

```python
def greet_all(names: list[str]) -> None:
  for name in names:
    print(f"Hello {name}")

names = ["Alice", "Bob", "Charlie"]
greet_all(names)

ages = [25, 35, 45]
greet_all(ages)
```

Running `mypy`

```bash
bad.py:10: error: Argument 1 to "greet_all" has incompatible type "list[int]";
expected "list[str]"  [arg-type]
```



---
level: 3
title: Example - Linting (1)
---

# Linting

```python
import numpy as np

def area(radius: float):
    return 3.14 * radius**2
```
<v-click>
```bash
bad.py:1:17: F401 [*] `numpy` imported but unused
  |
1 | import numpy as np
  |                 ^^ F401
  = help: Remove unused import: `numpy`
```
</v-click>

---
level: 3
title: Example - Linting (2)
---

# Linting (2)

```python
print("text.txt".strip(".txt"))
```
<v-click>
Surprising output
```python
"e"
```
</v-click>

<v-click>
```bash
B005    Using `.strip()` with multi-character strings is misleading
PLE1310 String `strip` call contains duplicate characters
```
</v-click>

---
level: 3
title: Example - Linting (3)
---

# Linting (3)

```python
greeting = "Hello, %" % "world"
```

Running `ruff`:

```bash
F501 `%`-format string has invalid format string: incomplete format
```

---
level: 3
title: Example - Refactoring As A Service 
---

# Refactoring as a Service 
<br>

````md magic-move
```python
if isinstance(hat, Sombrero) and hat.color == "green":
  wear(hat)

elif isinstance(hat, Sombrero) and hat.color == "red":
  put_away(hat)
```

```python
if isinstance(hat, Sombrero):
  if hat.color == "green":
    wear(hat)

  elif hat.color == "red":
    put_away(hat)
```
````

---
level: 3
title: Example - Linting for type checking 
zoom: 0.9
---

# Linting 🤝 type checking

```python
def add_three(x):
    return x+3
```
<br>
```bash
ANN201 Missing return type annotation for public function `add_three`
ANN001 Missing type annotation for function argument `x`
```

---
layout: end
hideInToc: true
---
# Fin
