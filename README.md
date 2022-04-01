# Priv-Page-2

[![ISC](https://img.shields.io/badge/License-ISC-blue.svg?style=flat-square)](https://en.wikipedia.org/wiki/ISC_license)
https://img.shields.io/github/license/StatSnips/priv-page-2

Serves static sites from a `privpage2` branch, using GitHub's OAuth2.

The server is written in [Crystal](https://crystal-lang.org/).

This open-source project is a fork of Priv-Page by Julien Reichardt. Priv-Page was discontinued in April 2022, and this project attempts to continue the service for its (small) user base.

## Features

- Designed for private Github repos (only)
- Serve by default a `privpage2` branch
- Supports branch prefixes, `privpage2-<MY_PREFIX>`

## Environment variables

| variable             | value       |
|----------------------|-------------|
|GITHUB_OAUTH_SECRET_ID|**mandatory**|
|GITHUB_OAUTH_CLIENT_ID|**mandatory**|
|PORT                  |    <any>    |

## Usage

1. First, set the environment variables. These are saved in the `.env-test` dot file.
2. Once updated, run: `set -a; . ./.env-test; set +a`
3. Build (add `--release` for release builds): `crystal build src/privpage.cr`
4. Execute: `./privpage`

## Architecture

For more information of how GitHub OAuth works, see [the official documentation](https://developer.github.com/apps/building-github-apps/identifying-and-authorizing-users-for-github-apps/).

## Serving from a documentation directory

GitHub Pages allows to serve from a `/docs` directory, which is not supported by Priv-Page-2.

However, it is possible to create a branch which will have the files of the directory at the root.

For a GitHub Actions example to how build a page site from a directory, [see this file](.github/workflows/documentation.yml).
Of course, adapt it to your needs.

For any question, add a comment to [the related issue](https://github.com/Priv-Page/privpage/issues/5).

## License
Copyright (c) 2022 Alvin G. Thomas - ISC License
Copyright (c) 2020-2021 Julien Reichardt - ISC License
