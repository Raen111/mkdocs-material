---
title: Getting started with Insiders
---

# Getting started with Insiders

Material for MkDocs Insiders is a compatible drop-in replacement for Material
for MkDocs, and can be installed similarly using [`pip`][pip],
[`docker`][docker] or [`git`][git]. Note that in order to access the Insiders 
repository, you need to [become an eligible sponsor] of @squidfunk on GitHub.

  [pip]: #with-pip
  [docker]: #with-docker
  [git]: #with-git
  [become an eligible sponsor]: index.md#how-to-become-a-sponsor

## Requirements

After you've been added to the list of collaborators and accepted the
repository invitation, the next step is to create a [personal access token] for
your GitHub account in order to access the Insiders repository programmatically 
(from the command line or GitHub Actions workflows):

1.  Go to https://github.com/settings/tokens
2.  Click on [Generate a new token]
3.  Enter a name and select the [`repo`][scopes] scope
4.  Generate the token and store it in a safe place

  [personal access token]: https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token
  [Generate a new token]: https://github.com/settings/tokens/new
  [scopes]: https://docs.github.com/en/developers/apps/scopes-for-oauth-apps#available-scopes

## Installation

### with pip

Material for MkDocs Insiders can be installed with `pip`:

``` sh
pip install git+https://${GH_TOKEN}@github.com/squidfunk/mkdocs-material-insiders.git
```

The `GH_TOKEN` environment variable must be set to the value of the personal
access token you generated in the previous step. Note that the personal access
token must be kept secret at all times, as it allows the owner to access your
private repositories.

### with docker

In case you want to use Material for MkDocs Insiders from within Docker, some
additional steps are necessary. While we cannot provide a hosted Docker image
for Insiders[^2], [GitHub Container Registry] allows for simple and
comfortable self-hosting:

1.  [Fork the Insiders repository]
2.  Enable [GitHub Actions] on your fork[^3]
3.  Create a new personal access token[^4]
    1.  Go to https://github.com/settings/tokens
    2.  Click on [Generate a new token]
    3.  Enter a name and select the [`write:packages`][scopes] scope
    4.  Generate the token and store it in a safe place
4.  Add a [GitHub Actions secret] on your fork
    1.  Set the name to `GHCR_TOKEN`
    2.  Set the value to the personal access token created in the previous step
5.  [Create a new release] to build and publish the Docker image
6.  Install [Pull App] on your fork to stay in-sync with upstream

The [`publish`][publish] workflow[^5] is automatically run when a new tag
(release) is created. When a new Insiders version is released on the upstream 
repository, the [Pull App] will create a pull request with the changes and
pull in the new tag, which is picked up by the [`publish`][publish] workflow
that builds and publishes the Docker image automatically to your private
registry.

Now, you should be able to pull the Docker image from your private registry:

```
docker login -u ${GH_USERNAME} -p ${GHCR_TOKEN} ghcr.io
docker pull ghcr.io/${GH_USERNAME}/mkdocs-material-insiders
```

Should you wish to add additional plugins to the insiders container image, follow the steps
outlined in the [Getting Started guide](../getting-started.md#with-docker).

  [^2]:
    Earlier, Insiders provided a dedicated Docker image which was available to
    all sponsors. On March 21, 2021, the image was deprecated for the reasons
    outlined and discussed in #2442. It was removed on June 1, 2021.

  [^3]:
    When forking a repository, GitHub will disable all workflows. While this
    is a reasonable default setting, you need to enable GitHub Actions to be
    able to automatically build and publish a Docker image on
    [GitHub Container Registry].

  [^4]:
    While you could just add the `write:packages` scope to the personal access
    token created to access the Insiders repository, it's safer to create a
    dedicated token which you'll only use for publishing the Docker image.

  [^5]:
    The Insiders repository contains two GitHub Actions workflows:

    - `build.yml` – Build and lint the project (disabled on forks)
    - `publish.yml` – Build and publish the Docker image

### with git

Of course, you can use Material for MkDocs Insiders directly from `git`:

```
git clone git@github.com:squidfunk/mkdocs-material-insiders.git mkdocs-material
```

The theme will reside in the folder `mkdocs-material/material`. When cloning
from `git`, the theme must be installed, so MkDocs can find the built-in
plugins:

```
pip install -e mkdocs-material
```

  [GitHub Container Registry]: https://docs.github.com/en/packages/guides/about-github-container-registry
  [Fork the Insiders repository]: https://github.com/squidfunk/mkdocs-material-insiders/fork
  [GitHub Actions]: https://docs.github.com/en/github/administering-a-repository/disabling-or-limiting-github-actions-for-a-repository
  [packages scope]: https://docs.github.com/en/developers/apps/scopes-for-oauth-apps#available-scopes
  [GitHub Actions secret]: https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository
  [Create a new release]: https://docs.github.com/en/github/administering-a-repository/managing-releases-in-a-repository#creating-a-release
  [Pull App]: https://github.com/apps/pull
  [publish]: https://github.com/squidfunk/mkdocs-material-insiders/blob/master/.github/workflows/publish.yml

## Upgrading

When upgrading Insiders, you should always check the version of Material for
MkDocs which makes up the first part of the version qualifier, e.g.Insiders
`4.x.x` is currently based on `8.x.x`:

```
8.x.x-insiders-4.x.x
```

If the major version increased, it's a good idea to consult the [upgrade
guide] and go through the steps to ensure your configuration is up to date and
all necessary changes have been made. If you installed Insiders via `pip`, you
can upgrade your installation with the following command:

```
pip install --upgrade git+https://${GH_TOKEN}@github.com/squidfunk/mkdocs-material-insiders.git
```

  [upgrade guide]: ../upgrade.md

## Built-in plugins

When you're using built-in plugins that are solely available via Insiders,
outside contributors won't be able to build your documentation project on their
local machine. This is the reason why we developed the [built-in group plugin]
that allows to conditionally load plugins.

  [^1]:
    Previously we recommended to use [configuration inheritance] to work around
    this limitations, but the brand new [built-in group plugin] is a much better
    approach, as it allows you to use a single configuration file for building
    your project with the community edition and Insiders version of Material
    for MkDocs.

  [built-in group plugin]: #built-in-group-plugin
  [configuration inheritance]: https://www.mkdocs.org/user-guide/configuration/#configuration-inheritance

### Built-in group plugin

[:octicons-tag-24: 9.3.0][Group plugin support] ·
:octicons-cpu-24: Plugin ·
:octicons-beaker-24: Experimental

The built-in group plugin adds support for conditionally loading plugins based
on environments. This makes enabling and disabling of multiple plugins much
simpler, as you can group them into logical units and enable or disable them
with an [environment variable]:

``` yaml
plugins:
  - group:
      enabled: !ENV CI
      plugins:
        - optimize
        - minify
```

[`enabled`](#+group.enabled){ #+group.enabled }

:   :octicons-milestone-24: Default: `false` – This option specifies whether
    the plugin is enabled when building your project. By default, the plugin is
    disabled, so you can use an [environment variable]:

    ``` yaml
    plugins:
      - group:
          enabled: !ENV CI
    ```

    Now, If you invoke MkDocs with that environment variable (or export the
    environment variable before invoking MkDocs), the plugin will be enabled:

    ``` sh
    CI=true mkdocs build
    ```

[`plugins`](#+group.plugins){ #+group.plugins }

:   :octicons-milestone-24: Default: _none_ – This option specifies the plugins
    that the group plugin should load when enabled. Note that the plugins must
    be specified as a list, even if there's only one plugin:

    ``` yaml
    plugins:
      - group:
          plugins:
            - optimize
            - minify
    ```

    The syntax is exactly the same as for all other plugins.

  [Group plugin support]: https://github.com/squidfunk/mkdocs-material/releases/tag/9.3.0
  [environment variable]: https://www.mkdocs.org/user-guide/configuration/#environment-variables
