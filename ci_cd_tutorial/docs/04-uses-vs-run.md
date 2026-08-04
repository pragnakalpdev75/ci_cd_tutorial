# Chapter 4: `uses:` vs `run:`

Almost every step in a workflow uses one of these two keywords:

```yaml
steps:
  - uses: ...

  - run: ...
```

Understanding the difference between them is the single most useful thing you can learn early on — once it clicks, most workflows become easy to read.

## `run:` means "execute this command"

This is the simpler of the two. GitHub literally runs whatever shell command you give it.

```yaml
- run: echo "Hello"
```

GitHub runs:

```bash
echo "Hello"
```

For a Django REST Framework project:

```yaml
- run: pip install -r requirements.txt
```

GitHub runs:

```bash
pip install -r requirements.txt
```

Nothing special is happening — it's simply executing shell commands, exactly as if you'd typed them into a terminal yourself.

## `uses:` means "download and execute an Action"

This is different. GitHub does **not** execute a shell command here.

```yaml
- uses: actions/checkout@v4
```

Instead, GitHub:

1. Downloads the Action.
2. Reads its `action.yml` definition.
3. Executes the program the Action defines.

Think of it like:

```python
import requests
```

You didn't write `requests` — someone else did — and you're using it. `uses:` works the same way: someone else wrote the automation, and you're reusing it.

## Can `run:` do everything?

Almost. You *could* write:

```yaml
- run: |
    git clone ...

# or
- run: |
    curl ...

# or
- run: |
    docker build ...
```

GitHub doesn't care — it simply executes shell commands either way.

## So why use Actions at all?

Imagine every project had to write this out by hand:

```text
git clone ...
git checkout ...
git fetch ...
git config ...
authentication...
submodules...
LFS...
```

Every project would end up copy-pasting hundreds of lines just to check out a repository correctly (including edge cases like submodules, LFS, and auth). Instead:

```yaml
- uses: actions/checkout@v4
```

One line. Millions of repositories reuse it, and the Action's maintainers handle the edge cases for you.

## Inputs (`with:`)

Many Actions accept configuration through `with:`, similar to passing arguments to a function.

```yaml
# Example
- uses: actions/setup-python@v5
  with:
    python-version: "3.11"

# This is similar to calling a function:
# setup_python(version="3.11")

# Another example
- uses: actions/upload-artifact@v4
  with:
    name: reports
    path: reports/

# Equivalent idea:
# upload_artifact(name="reports", path="reports/")
```

## The key takeaway

Whenever you look at a workflow step, ask yourself:

- **Is this step using an existing tool?** → It probably uses `uses:`.
- **Is this step executing commands I could type into a terminal?** → It probably uses `run:`.

Once you can answer those two questions instinctively, most GitHub Actions workflows become much easier to understand.
