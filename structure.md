# Project Structure

## Overview
- (Directory) .github
    - `ci.yml`
    - `documentation.yml`
- (Directory) bin
- (Directory) docs
- (Dictorory) lib
- (Directory) spec
    - `session_spec.cr`
    - `user_repository_spec.cr`
- (Directory) src
    - (Directory) github
        - `oauth.cr`
        - `session_data.cr`
    - `github.cr`
    - `privpage.cr`
    - `session.cr`
    - `user_repository.cr`
- `.env_test`
- `.gitignore`
- `LICENSE`
- (Executible) `privpage`
- `README.md`
- `shard.yml`
- `structure.md`

## .github (Workflows)

This directory enables [Github Workflows](https://docs.github.com/en/actions/using-workflows). Right now there are two workflows as seen by their YAML files.
- Continuous Integration (CI)
- Documentation

### CI

The [CI](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration) integration activations on a Github push action (within the private respository using Priv-Page-2). This action runs on `ubuntu-latest`. It calls the latest Alpine dockerimage from [DockerHub](https://hub.docker.com/r/crystallang/crystal/tags?page=1&ordering=-name&name=latest). Alpine is slightly smaller than Ubuntu crystal containers and also utilizes the `musl-libc` library instead of `gnu-libc` as discussed [here](https://crystal-lang.org/2020/02/02/alpine-based-docker-images.html). 

CI downloads the private repo's source code using Github's [checkout](https://github.com/actions/checkout) action (v3). CI then checks for proper formatting (using `crystal tool`), installs shards (creating a `shard.lock` file), compiles and run specs (using the spec directory), and builds shards. For those new to Crystal (like me), `shards` is the dependency manager for Crystal. The shards repo is [here](https://github.com/crystal-lang/shards). This command requires a `shard.yml` file in the working directory. The `shard.lock` file ensures that the *same* version of dependencies are installed when `shards install` or `shards build` is run in the future. Thus, if you delete the `shard.lock` file then you may be running with newer version of the dependencies.

In summary, CI actives every time Github detects a push action on the private repo. It looks at the changes then prepares to compile the shards and app executable.

### Documentation

Like CI above, this github workflow actives on a Github repo push. It also utilizes the latest Alpine dockerimage of Crystal running on `ubuntu-latest`. The purpose of this workflow is to create API documentation using [`crystal docs`](https://crystal-lang.org/reference/1.3/using_the_compiler/index.html#crystal-docs). This was not used/active when Priv-Page-2 was forked from Priv-Page.

After running `crystal docs`, this workflow updates, commits, and pushes changes to the documentation.

## bin
Built by the CI Github workflow.

## docs
Built by the Documentation Github workflow.

## lib
Built by the CI Github workflow.

## specs
This directory contains two spec files run during the CI Github workflow. The goal of these files are to assess the build quality of `session` and `user_repository` (discussed later). The Crystal guide for testing is [available](https://crystal-lang.org/reference/1.3/guides/testing.html) for those interested.

### session_spec.cr
Spec file for `session`. Requires: (1) `spec` and (2) `../src/session`.

### user_repository_spec.cr
Spec file for `user_repository`. Requires: (1) `spec` and (2) `../src/user_repository`.

## src
This directory contains the key `.cr` files for the application. 

### github
This directory contains two `.cr` files used by the `scr/github.cr`.

#### oauth.cr
Requires `random/secure`, `oauth2`, and `httep/client`. All three are core shards included in the Crystal installation. The location of these shards should be in your PATH environment variable. For instance, on Ubuntu via WSL2 the shards can be found in the `/usr/share/crystal/src` directory.

In the Github `module`, the app captures the environment variables described in `.env-test`. The function `self.request_identity_redirect` takes two inputs: `state` and `response`.

The `struct` State "protects against cross-site request forgery attacks." It contains the functions `initialize`, `get_access_token`, `value`, and `self.from_string`. Briefly, this `struct` assess the valid state of a Github access token. If a client successfully passes Github's authorization process, then the app will be notified that the client presented a valid token and has proper authorization to visit the page.

#### session_data.cr
Requires `http/client` and `json`. These are core shards in the Crystal installation.

The `struct` GitHub::SessionData get the content of the path to the private repo. First it assess whether the repo is private. If it is, it then looks for a `index.html` file. The application is also able to process `.md` and `.rst` files. For other types of files, the app relies on Github's `vnd` application to present the media (see more [here](https://docs.github.com/en/rest/overview/media-types)).

If the repo is somehow invalid, then the `struct` will return that it is an invalid repository. If the repo is public, then the `struct` will redirect to the Github forbidden page.

### github.cr
Requires `http_client`, `./session`, `./user_repository`, `./github/oauth`, and `./github/session_data`. The first is a core shard and the others are described elsewhere in this document.

The main module, `GitHub`, follows the principles of Github's [OAuth web flow](https://developer.github.com/apps/building-github-apps/identifying-and-authorizing-users-for-github-apps/). It creates a `PrivPage2` `Session` and starts the session garbage collector in the background.

The main functions are `handle_request`, `handle_callback`, and `redirect`. `handle_callback` is called if `handle_request` receives the string "callback" as the first part of the subdomain (i.e., the private repo being served). If the user has a valid, authenticated session, then `handle_request` will present the content of the private repo. Otherwise, the user may be instructed to login (if they are timed out). `redirect` will redirect the client to the location of the private repo's served webpage and/or file.

### privpage2.cr
Requires `http/server`, `log`, `./user_repository`, and `./github`. The first two are core shards and the latter two are described below. This shard describes the core module `PrivPage2`. 

The main functions are `proc` and `start`. `proc` features security functions such as cross-site filter (XSS), MIME-sniffing, and clickjacking protection. This is also where OAuth is called (only GitHub at this time). `start` starts the PrivPage2 server and sends it out locally (by default, to port 3000). 

### session.cr
Requires `log` which is a core shard. Session creates a `PrivPage2` `Session` `struct`. It has the functions `initialize`, `size`, `add`, `start_gc`, and `get?`. The shard gets a session, if present, and stores it. Sessions older than `max_period` are cleaned by the garbage collector.

### user_repository.cr
Requires `http/server/response` which is a core shard. The core `struct` transforms the content of a private repo into a safe subdomain. The default delimited for URLs is `--`. This delimiter spaces the user name, repository, and (optinoally) branch. For using the branch feature, users must create branches prefixed by `privpage2-`. The minimum requirement is one branch called `privpage2`; if there is not such a branch, one will be made by the `full_branch` function.

Within `struct` `PrivPage2::UserRepository` are the functions `full_branch`, `initalize`, `self.from_subdomain`, and `self.split_first_subdomain_part`. Together, these functions paste together a valid URL features the user, repo name, and (optionally) branch. Repos that contain the delimiter `--` or `.` will return errors.

## .env-test
This file stores environmental variables necessary to run the application. Specifically, this app requires two variables from GitHub: (1) `GITHUB_OAUTH_SECRET_ID` and (2) `GITHUB_OAUTH_CLIENT_ID`. There is a third (optional) environment variable that describes the port when testing on your localhost. 

## .gitignore

Git will ignore the bin and lib directories, the main executable, and the pem file (private key).

## LICENSE
States the type of license.

## README.md
A README which describes the repo for Github.

## shard.lock
Created by shart `shard install`.

## shard.yml
This YAML file follows the style guidelines recommend by Crystal (and checked for by `crystal tool`). It describes the app `name`, `authors`, `targets`, `development_dependencies`, and `license`.

## structure.md
The present file. This markdown file desribes the components of the Priv-Page-2 app.