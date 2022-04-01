# Priv-Page-2

![GitHub](https://img.shields.io/github/license/StatSnips/priv-page-2)

Serves static sites from a `privpage2` branch, using GitHub's OAuth2.

The server is written in [Crystal](https://crystal-lang.org/).

This open-source project is a fork of Priv-Page by Julien Reichardt. Priv-Page was discontinued in April 2022, and this project attempts to continue the service for its (small) user base.

## Features

- Designed for private Github repos (only)
- Serve by default a `privpage2` branch
- Supports branch prefixes, `privpage2-<MY_PREFIX>`

## Architecture

For more information of how GitHub OAuth works, see [the official documentation](https://developer.github.com/apps/building-github-apps/identifying-and-authorizing-users-for-github-apps/).

## Serving from a documentation directory

GitHub Pages allows to serve from a `/docs` directory, which is not supported by Priv-Page-2.

However, it is possible to create a branch which will have the files of the directory at the root. For a GitHub Actions example to how build a page site from a directory, [see this file](.github/workflows/documentation.yml).

## License
Copyright (c) 2022 Alvin G. Thomas - ISC License
Copyright (c) 2020-2021 Julien Reichardt - ISC License
