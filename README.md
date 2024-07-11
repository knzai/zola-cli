# GH Action for Zola CLI

## Deprecated for: https://github.com/zolacti/on

![Build Status](https://img.shields.io/github/actions/workflow/status/knzai/zola-cli/test.yml)

A GitHub action that replicates the [zola](https://github.com/getzola/zola) static site generator's [CLI](https://www.getzola.org/documentation/getting-started/cli-usage/) as much as makes sense in a GH Action (no serve or init). 

You probably want [zola-build](https://github.com/knzai/zola-build) instead for the full standard deploy. This is for calling individual commands for more flexibility.

## Setup

- If you want to push this content to GH Pages, make sure that workflows have read *and write* permissions in the repos **Settings > Actions > General > in Workflow permissions**
- Also, after a push as been made pages branch (generally gh-pages) **Settings > General > Pages > Build and deployment** should be set to "deploy from a branch" and the branch you using for pages. GH's [internal action](https://github.com/actions/deploy-pages) will then handle pushing the artifact up and publishing.

## Note

Each action (build and check) live in their own branch so are called with the @ref syntax, eg `knzai/zola-cli@build`. If you just run the default `knzai/zola-cli@v2` zola will be installed for you but no other actions run.

## Variables
Variables depend on which command you are calling and matches the flags and usage in the Zola CLI

### knzai/zola-cli@build
```yml
inputs:
  root:
    description: Directory to use as root of project 
    required: false
    default: '.'
    type: string
  config:
    description: Path to a config file other than config.toml in the root of project
    required: false
    default: 'config.toml'
    type: string
  base-url:
    description: Force the base URL to be that value (defaults to the one in config)
    required: false
    default: ''
    type: string
  drafts:
    description: Include drafts when loading the site
    required: false
    default: false
    type: boolean
  output-dir:
    description: Outputs the generated site in the given path (by default 'public' dir in project root)
    required: false
    default: 'public'
    type: string
  force:
    description: Force building the site even if output directory is non-empty
    required: false
    default: false
    type: boolean
```

### knzai/zola-cli@check
```yml
inputs:
  root:
    description: Directory to use as root of project 
    required: false
    default: '.'
    type: string
  config:
    description: Path to a config file other than config.toml in the root of project
    required: false
    default: 'config.toml'
    type: string
  drafts:
    description: Include drafts when loading the site
    required: false
    default: false
    type: boolean
```


## Examples

Basic standard deploy, just use [zola-build](https://github.com/knzai/zola-build)
```yml
name: Deploy Zola to GH Pages
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
      with:
        submodules: true
    - uses: knzai/zola-build@main
```

If you want to be able to run commands seperately for more flexibility then call this action with the appropriate @

Checks out out a different branch for the content, without submodules; skips the check by only running build; includes drafts; and passes some additional flags to the deploy (erasing history on the non standard branch it deploys to). This still depends on the default GH action to publish, unless you include an action to push the artifacts yourself.


```yml
name: Deploy Zola to GH Pages
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
      with:
        ref: site
    - uses: knzai/zola-cli@build 
      with:
        drafts: true
        root: docs
        output-dir: not-public
    - name: Deploy 🚀
      uses: JamesIves/github-pages-deploy-action@v4
      with:
        folder: not-public
        single-commit: true
        branch: not-gh-pages
```

Deploy to gh-pages branch on a push to the main branch and it will build and check external links on pull requests
```yml
name: Deploy Zola to GH Pages
on:
  push:
    branches:
      - main 
  pull_request:
  
jobs:
  build:
    runs-on: ubuntu-latest
    if: github.ref != 'refs/heads/main'
    steps:
      - uses: actions/checkout@master
      - uses: knzai/zola-cli@check
      - uses: knzai/zola-cli@build
          
  build_and_deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@master
      #If doing default params the following be replaced by [zola-build](https://github.com/knzai/zola-build)
      - uses: knzai/zola-cli@check
      - uses: knzai/zola-cli@build
      - name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@v4
```


## Development

The test site lives in the `site` branch.

The install, build, and test commands all live in their eponymous branches.


## Acknowledgements

This project is a simplification of [my earlier version](zola-deploy-action) removing the GH Pages deploy logic to use [a more robust existing actions for that](JamesIves/github-pages-deploy-action), before I decided to separate the CLI and deploy logic fully. My earlier version was a itself a port of [Shaleen Jain's Dockerfile based Zola Deploy Action](shalzz/zola-deploy-action) over to a composite action.

##
