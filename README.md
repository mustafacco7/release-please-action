# Release Please Action

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)

Automate releases with Conventional Commit Messages.

## Setting up this action

1. If you haven't already done so, create a `.github/workflows` folder in your
  repository (_this is where your actions will live_).
2. Now create a `.github/workflows/release-please.yml` file with these contents:

   ```yaml
    on:
      push:
        branches:
          - main
    name: release-please
    jobs:
      release-please:
        runs-on: ubuntu-latest
        steps:
          - uses: GoogleCloudPlatform/release-please-action@v2.0.0
            with:
              token: ${{ secrets.GITHUB_TOKEN }}
              release-type: node
              package-name: release-please-action
    ```

3. Merge the above action into your repository and make sure new commits follow
  the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
  convention, [release-please](https://github.com/googleapis/release-please)
  will start creating Release PRs for you.

## Configuration

| input | description |
|:---:|---|
| `token` | A GitHub secret token, you will most likely want to use the special `secrets.GITHUB_TOKEN` |
| `release-type` | What type of project is this a release for? Reference [Release types supported](#release-types-supported); new types of releases can be [added here](https://github.com/googleapis/release-please/tree/master/src/releasers) |
| `package-name` | A name for the artifact releases are being created for (this might be the `name` field in a `setup.py` or `package.json`) |
| `bump-minor-pre-major` | Should breaking changes before 1.0.0 produce minor bumps?  Default `No` |
| `--path`          | create a release from a path other than the repository's root |
| `--monorepo-tags` | add prefix to tags and branches, allowing multiple libraries to be released from the same repository. |

| output | description |
|:---:|---|
| `release_created` | `true` if the release was created, `false` otherwise |
| `upload_url` | Directly related to [**Create a release**](https://developer.github.com/v3/repos/releases/#response-4) API |
| `tag_name` | Directly related to [**Create a release**](https://developer.github.com/v3/repos/releases/#response-4) API |

### Release types supported

Release Please automates releases for the following flavors of repositories:

| release type | description |
|:---:|---|
| `node` | [A Node.js repository, with a package.json and CHANGELOG.md](https://github.com/yargs/yargs) |
| `python` | [A Python repository, with a setup.py, setup.cfg, and CHANGELOG.md](https://github.com/googleapis/java-storage) |
| `ruby` | [A Ruby repository, with version.rb and CHANGELOG.md](https://github.com/google/google-id-token) |
| `terraform-module` | [A terraform module, with a version in the README.md, and a CHANGELOG.md](https://github.com/terraform-google-modules/terraform-google-project-factory) |
| `simple` | [A repository with a version.txt and a CHANGELOG.md](https://github.com/googleapis/gapic-generator) |

## How release please works

Release Please automates CHANGELOG generation, the creation of GitHub releases,
and version bumps for your projects. Release Please does so by parsing your
git history, looking for [Conventional Commit messages](https://www.conventionalcommits.org/),
and creating release PRs.

### What's a Release PR?

Rather than continuously releasing what's landed to your default branch,
release-please maintains Release PRs:

<img width="400" src="/screen.png">

These Release PRs are kept up-to-date as additional work is merged. When you're
ready to tag a release, simply merge the release PR.

### How should I write my commits?

Release Please assumes you are using [Conventional Commit messages](https://www.conventionalcommits.org/).

The most important prefixes you should have in mind are:

* `fix:` which represents bug fixes, and correlates to a [SemVer](https://semver.org/) 
  patch.
* `feat:` which represents a new feature, and correlates to a SemVer minor.
* `feat!:`,  or `fix!:`, `refactor!:`, etc., which represent a breaking change
  (indicated by the `!`) and will result in a SemVer major.

## Automating publication to npm

With a few additions, the Release Please action can be made to publish to
npm when a Release PR is merged:

```yaml
on:
  push:
    branches:
      - main
name: release-please
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: GoogleCloudPlatform/release-please-action@v2.0.0
        id: release
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          release-type: node
          package-name: test-release-please
      # The logic below handles the npm publication:
      - uses: actions/checkout@v2
        # these if statements ensure that a publication only occurs when
        # a new release is created:
        if: ${{ steps.release.outputs.release_created }}
      - uses: actions/setup-node@v1
        with:
          node-version: 12
          registry-url: 'https://registry.npmjs.org'
        if: ${{ steps.release.outputs.release_created }}
      - run: npm ci
        if: ${{ steps.release.outputs.release_created }}
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{secrets.NPM_TOKEN}}
        if: ${{ steps.release.outputs.release_created }}
```

> So that you can keep 2FA enabled for npm publications, we recommend setting
`registry-url` to your own [Wombat Dressing Room](https://github.com/GoogleCloudPlatform/wombat-dressing-room) deployment.

## License

Apache Version 2.0

## Examples of Usage

Here are some examples of how to use the Release Please action in different scenarios:

### Example 1: Basic Usage

```yaml
on:
  push:
    branches:
      - main
name: release-please
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: GoogleCloudPlatform/release-please-action@v2.0.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          release-type: node
          package-name: release-please-action
```

### Example 2: Using a Custom Path

```yaml
on:
  push:
    branches:
      - main
name: release-please
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: GoogleCloudPlatform/release-please-action@v2.0.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          release-type: node
          package-name: release-please-action
          path: custom/path
```

### Example 3: Monorepo Tags

```yaml
on:
  push:
    branches:
      - main
name: release-please
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: GoogleCloudPlatform/release-please-action@v2.0.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          release-type: node
          package-name: release-please-action
          monorepo-tags: true
```

## How to Contribute

We welcome contributions to the Release Please action! To contribute, please follow these steps:

1. Fork the repository and clone it to your local machine.
2. Create a new branch for your feature or bugfix.
3. Make your changes and commit them with clear and concise commit messages.
4. Push your changes to your forked repository.
5. Open a pull request with a description of your changes.

### Reporting Issues

If you encounter any issues or have questions, please open an issue in the repository. Provide as much detail as possible to help us understand and resolve the issue.

### Submitting Pull Requests

When submitting a pull request, please ensure that your changes are well-documented and include tests if applicable. Follow the existing code style and conventions.

Thank you for your contributions!
