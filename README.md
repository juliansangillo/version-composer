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
See [action.yml]()
## How it knows your latest version?
In order to create the next version, it needs to know what the latest version is. It does this by getting the latest annotated tag on your current branch that was created in your repo. It then strips off the 'v' character that is prepended to the tag. This string should be numbers separated by periods.
## Revision Patterns
It uses a string pattern to determine the structure of your version. There are 4 values max that can be used in your version and they can be in any order: major revision, minor revision, build number, and patch number. These values are all integers separated by periods. The default pattern is "M.m.b".
###
* M - major
* m - minor
* b - build
* p - patch
### Examples
* M.m.b
* M.m.b.p
* M.m
* M.m.p.b
* m.M.b (Not standard but the action will allow the values to be in any order)
## Branching Strategy
Version Composer assumes that there are three branches in your repo with each branch representing a stage of development. There should be a development branch, a test branch, and a stable branch. The development branch is the most unstable branch and is used to release new work that has yet to be tested. The test branch is for features ready to be tested. The stable branch is for final production releases. These could be customized to be any branch. The defaults for dev, test, and stable are develop, test, and master branches respectively.
###
new version -> push -> develop
develop -> PR -> test
test -> PR -> master
## Pushing new build to dev branch
It will always use the latest commit that was pushed to determine how to create the new revision, no matter how many commits were made locally. A standard commit message will increment your version's build number. This resets the patch number. (Note: all examples from here on assume a pattern of M.m.b.p)
### Example
```
git commit -m "Add file foo.txt"
```
###
1.0.7.5 -> 1.0.8.0
## Pushing new minor revision
A commit message with the substring '\[minor\]' will increment your version's minor revision. This resets the build and patch. This tag could exist anywhere in your message and it will still be read. For the sake of example, we will put the tag at the end of the message.
### Example
```
git commit -m "Update dependencies [minor]"
```
###
2.3.5.2 -> 2.4.0.0
## Pushing new major revision
A commit message with the substring '\[major\]' will increment your version's major revision. This resets the minor, build, and patch. This string could exist anywhere in your message and it will still be read.
### Example
```
git commit -m "Add fancy new UI [major]"
```
###
1.5.10.13 -> 2.0.0.0
## Pushing new patch
A commit message with the substring '\[patch\]' will increment your version's patch number. This string could exist anywhere in your message and it will still be read. Unlike the other values, patch is a bit special. It doesn't increase by just one. It increases by the number of commits that were pushed since the last known version.
### Example
```
git commit -m "Something 1"
git commit -m "Something 2"
git commit -m "Something 3"
git commit -m "Something 4"
git commit -m "Something 5 [patch]"
```
###
3.0.1.0 -> 3.0.1.5
## Release Tags
The release tag is a string of capitalized ASCII characters marking the version as a type of release based on branch (example: 1.0.0.TEST). Using release tags are advisable since multiple branches are in play here. It is possible to customize what tags are used for which branches. By default, the dev branch uses the 'UNSTABLE' tag, the test branch uses the 'TEST' tag, and the stable branch uses no tag. You can enter an empty string for no tag. It is recommended though that all branches have different tags, otherwise there will be conflicting versions.
### What about versioning on test and stable branches?
When you merge a pull request (or push into test or production branches), the version is not incremented as you are not introducing a new instance of your project. Instead, you are promoting an already existing version to a higher branch. The only thing that changes in this case is the release tag. Below examples show the flow when the latest version is 2.1.0 and a new build is pushed.
### local -> develop
2.1.1.UNSTABLE
### develop -> test
2.1.1.TEST
### test -> master
2.1.1
## Ouputs
* **version**: The new version that was created with the release tag appended
* **raw-version**: The new version that was created without the release tag appended
* **is-prerelease**: Is true if this is a release into a non-production environment (dev or test) and indicates the build may be unstable. Returns false otherwise.
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
