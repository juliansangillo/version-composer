# Version Composer v1
Creates a new version identifier to use with your builds and releases.
## Features
* Return as output a version with and without a release tag appended at the end
* Control how your versions are formatted
* Choose what branches you want new versions to be based off of at each stage of development
* Choose what release tags to use at each stage of development
## Version Format
\<your-version\>\[.\<release-tag\>\]
### Examples
* 2.3
* 3.2.15
* 2.3.TEST
* 3.2.15.UNSTABLE
# Usage
See [action.yml](https://github.com/juliansangillo/version-composer/blob/master/action.yml)
## How it knows your latest version?
In order to create the next version, it needs to know what the latest version is. It does this by getting the latest annotated tag in your repo. It then strips off the 'v' character that is prepended to the tag. This string should be numbers separated by periods.
## Revision Patterns
It uses a string pattern to determine the structure of your version. There are 4 values that may be used in your version and they can be in any order: major revision, minor revision, build number, and patch number. These values are all integers separated by periods. The default pattern is "M.m.b".
###
* M - major
* m - minor
* b - build
* p - patch
### Examples
* `M.m.b`
* `M.m.b.p`
* `M.m`
* `M.m.p.b`
* `m.M.b` (Not a standard version but the action will allow the values to be in any order)
## Branching Strategy
Version Composer assumes that there are three branches in your repo with each branch representing a stage of development. There should be a development branch, a test branch, and a stable branch. The development branch is the most unstable branch and is used to release new work that has yet to be tested. The test branch is for features ready to be tested. The stable branch is for final production releases. These could be customized to be any branch. The defaults for dev, test, and stable are develop, test, and master branches respectively.
###
push: new version -> develop  
PR: develop -> test  
PR: test -> master
## Build vs Patch
Patch is different than the other revisions. Build is always incremented by 1. It identifies that particular build within the minor revision. Patch, however, is incremented by the number of commits that were pushed to this new version. So if there are 5 new commits, then patch will be incremented by 5. Major and minor are also always increased by 1. You can use the revision-pattern and the preferred-type inputs to change whether you prefer build or patch revisions for changes where you don't explicitly specify. More on preferred-type below.
## Version Bumping
There are 4 different revision types that determine how a version is incremented when a new one is created in the development branch. They are: major, minor, build, and patch. The revision type during execution is based on the commit messages being included since the latest version. It is assumed your commit messages are following the [Angular Commit Message Conventions](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines) with the addition of 'build' and 'patch' types. Higher priority changes always take precedence. Major changes are always more important than minor ones. Likewise, minor is higher than build and patch, while build and patch are about the same. If none of the commit messages below exist, then it will fall back to the preferred-type. The preferred-type can be either build or patch and changing it will change whether you prefer to use the build value or the patch value for most types of changes. This can be overwridden during execution by specifying build or patch in one of your commits. By default preferred-type is build.
| Revision<br>Type | Commit Message |
|-|-|
| Patch <br>Revision | `patch: stop graphite breaking when too much pressure applied`<br>`patch(pencil): stop graphite breaking when too much pressure applied` |
| Build<br>Revision | `build: stop graphite breaking when too much pressure applied`<br>`build(pencil): stop graphite breaking when too much pressure applied` |
| Minor<br>Revision | `feat: add 'graphiteWidth' option`<br>`feat(pencil): add 'graphiteWidth' option`<br><br>`perf: remove graphiteWidth option`<br>`perf(pencil): remove graphiteWidth option` |
| Major<br>Revision | `BREAKING CHANGE: The graphiteWidth option has been removed.<br>The default graphite width of 10mm is always used for performance reasons.` |
## Release Tags
The release tag is a string of capitalized ASCII characters marking the version as a type of release based on branch (example: 1.0.0.TEST). Using release tags are advisable since multiple branches are being used here. It is possible to customize what tags are used for which branches. By default, the dev branch uses the 'UNSTABLE' tag, the test branch uses the 'TEST' tag, and the stable branch uses no tag. You can enter an empty string for no tag. It is recommended though that all branches have different tags, otherwise there will be conflicting versions.
### What about versioning on test and stable branches?
When you merge a pull request (or push into test or production branches), the version is not incremented as you are not introducing a new instance of your project. Instead, you are promoting an already existing version to a higher branch. The only thing that changes in this case is the release tag. Below examples show the flow when the latest version is 2.1.0 and a new build is pushed.
### local -> develop
`2.1.1.UNSTABLE`
### develop -> test
`2.1.1.TEST`
### test -> master
`2.1.1`
## Inputs
### revision-pattern
Custom pattern showing version structure. Default is `M.m.b`
### preferred-type
The preferred revision type to fallback to if no other type can be determined. Possible values are 'build' or 'patch'. Default is `build`
### stable-branch
Name of branch in your repo that will be used for stable releases. Default is `master`
### test-branch
Name of branch in your repo that will be used for test releases. Default is `test`
### dev-branch
Name of branch in your repo that will be used for development releases. Default is `develop`
### stable-tag
Custom release tag name for stable releases. By default, stable releases have no stable tag.
### test-tag
Custom release tag name for test releases. Default is `TEST`
### dev-tag
Custom release tag name for development releases. Default is `UNSTABLE`
## Ouputs
### version
The new version with the release tag appended at the end.
### raw-version
Version without the release tag at the end.
### is-prerelease
True if this version is an unstable release (dev or test branch) or false otherwise.
## Setup
Below are example workflows using either the defaults or entering custom values. It is also worth noting that this action simple calculates and returns the version id among other values. It doesn't push to your repository in any way. This allows you to use the output in multiple places throughout your workflow, including in actions that will change your repo.
### Workflow 1
```yml
name: Workflow1
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Version Composition
        id: composer
        uses: juliansangillo/version-composer@v1

      - name: Create Release
        id: release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ steps.composer.outputs.version }}
          release_name: Release ${{ steps.composer.outputs.raw-version }}
          draft: false
          prerelease: ${{ steps.composer.outputs.is-prerelease == 'true' }}
```
### Workflow 2
```yml
name: Workflow2
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Version Composition
        id: composer
        uses: juliansangillo/version-composer@v1
        with:
          revision-pattern: M.m.b.p

      - name: Create Release
        id: release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ steps.composer.outputs.version }}
          release_name: Release ${{ steps.composer.outputs.raw-version }}
          draft: false
          prerelease: ${{ steps.composer.outputs.is-prerelease == 'true' }}
```
### Workflow 3
```yml
name: Workflow3
on: push
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Version Composition
        id: composer
        uses: juliansangillo/version-composer@v1
        with:
          revision-pattern: M.m.b.p
          preferred-type: patch
          stable-branch: release
          test-branch: uat
          dev-branch: dev
          stable-tag: STABLE
          test-tag: TEST
          dev-tag: null

      - name: Create Release
        id: release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ steps.composer.outputs.version }}
          release_name: Release ${{ steps.composer.outputs.raw-version }}
          draft: false
          prerelease: ${{ steps.composer.outputs.is-prerelease == 'true' }}
```
## See Also
[juliansangillo/version-maestro](https://github.com/marketplace/actions/version-maestro)
