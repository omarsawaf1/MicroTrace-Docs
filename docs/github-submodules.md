# How To Use Git Submodules

This project keeps the documentation site as a **separate repository** inside the main codebase by using a git submodule.

Current submodule example in MicroTrace:

```text
docs-site/ -> https://github.com/omarsawaf1/MicroTrace-Docs.git
```

## Why Use a Submodule

Submodules are useful when:

- you want separate git history
- you want a repo to live inside another repo
- you do not want to copy documentation files into the main project

## Clone a Repository With Submodules

```bash
git clone --recurse-submodules <repo-url>
```

## Initialize Submodules After a Normal Clone

```bash
git submodule update --init --recursive
```

## Pull the Latest Docs Changes

```bash
git submodule update --remote --merge docs-site
```

## Edit the Submodule

1. Enter the submodule directory:

   ```bash
   cd docs-site
   ```

2. Create or edit files as normal
3. Commit inside the submodule
4. Return to the main repository
5. Commit the updated submodule pointer in the main repository

## Common Workflow

```bash
cd docs-site
git checkout main
git pull
```

Make documentation changes, then:

```bash
git add .
git commit -m "Update docs"
git push
```

Then in the parent repository:

```bash
cd ..
git add docs-site
git commit -m "Update docs-site submodule pointer"
```

## Important Rule

!!! warning
    A submodule is not stored as a normal folder in the parent repository. The parent repository stores a pointer to a specific commit of the child repository.

## Troubleshooting

### Submodule looks empty

Run:

```bash
git submodule update --init --recursive
```

### Submodule is on detached HEAD

Inside the submodule:

```bash
git checkout main
```

Then pull or commit normally.
