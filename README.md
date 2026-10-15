# devenv configurations

This repository stores shared [devenv](https://devenv.sh/) configurations for GitHub repositories.

Each configuration uses the GitHub path of its target repository. For example:

```text
rails/rails/devenv.nix
rails/rails/devenv.yaml
```

The configuration above applies to `rails/rails`.

## Requirements

Install these tools before you use the helper:

- [Git](https://git-scm.com/)
- [devenv](https://devenv.sh/)
- Z shell (`zsh`)

The helper stores a local copy of this repository at:

```text
$HOME/src/github.com/rafaelfranca/devenv
```

## Install `devenv-allow`

Add this line to `~/.zshrc`:

```zsh
source "$HOME/src/github.com/rafaelfranca/devenv/devenv-allow.sh"
```

Reload the function in the current shell:

```zsh
source ~/.zshrc
```

## Use `devenv-allow`

Run the helper from a target repository:

```zsh
cd ~/src/github.com/rails/rails
devenv-allow
```

With no argument, the helper reads the target repository's `origin` remote. It accepts these GitHub remote forms:

- `https://github.com/org/repo.git`
- `git@github.com:org/repo.git`
- `ssh://git@github.com/org/repo.git`

You can pass the repository path explicitly:

```zsh
devenv-allow rails/rails
```

The helper then:

1. Clones the central repository if it does not exist locally.
2. Updates an existing clean checkout with a fast-forward-only merge.
3. Loads `$HOME/src/github.com/rafaelfranca/devenv/org/repo/devenv.nix`.
4. Copies `devenv.yaml` temporarily when the central configuration provides one.
5. Runs `devenv allow --from path:<configuration-path>`.
6. Removes the temporary `devenv.yaml` after `devenv allow` finishes.

The helper does not change an existing `devenv.yaml` in the target repository.

## Repository layout

Add a target configuration at `<org>/<repo>`:

```text
<org>/<repo>/devenv.nix
<org>/<repo>/devenv.yaml   # optional
```

The target repository must have a `devenv.nix` file. The helper refuses paths that contain more than one slash, paths that start with `/`, and paths that end with `/`.

## Roundhouse

The `rubys/roundhouse` configuration includes `libyaml` and `pkg-config`. Bundler uses them to build the native extension for `psych`.

It also includes:

- `libclang`, with `LIBCLANG_PATH` set. Bindgen needs it to build `ruby-rbs-sys` and `ruby-prism-sys`.
- `vips`, with its library directory on `LD_LIBRARY_PATH`. The `ruby-vips` gem loads `libvips.so.42` at runtime.

Run `devenv tasks run roundhouse:gems` to install the gems that Roundhouse's CI unit job installs for emitted-Ruby tests: `rails-html-sanitizer`, `sqlite3`, `bcrypt`, `rack`, `ruby-vips`, `markly`, and Active Record 8.1. The task does nothing once every one of them loads.

## Writebook

The `basecamp/writebook` configuration uses Ruby `3.4.7`, SQLite, libvips, and Redis. It includes `libyaml` and `pkg-config` to build native gems.

The setup task installs gems and prepares the database. It uses the Ruby version from devenv, not `mise` from the application's `bin/setup`.

1. Run these commands from the Writebook directory:

   ```zsh
   devenv-allow
   devenv tasks run writebook:setup
   devenv up -d
   devenv shell bin/dev
   ```

2. Open `http://localhost:3007`.

3. Stop the server with `Ctrl-C`. Stop Redis with:

   ```zsh
   devenv down
   ```

Development Action Cable connects to Redis on port `6379`. This configuration binds Redis to `127.0.0.1:6379` without automatic port allocation.

## Fizzy

The `basecamp/fizzy` configuration uses Ruby `3.4.8`, SQLite, libvips, ImageMagick, OpenSlide, FFmpeg, and MuPDF. It includes `libyaml` and `pkg-config` to build native gems.

The setup task installs gems and prepares the database. Rails adds sample data when it creates the database. The task does not run the application's `mise` setup.

1. Run these commands from the Fizzy directory:

   ```zsh
   devenv-allow
   devenv tasks run fizzy:setup
   devenv shell bin/dev
   ```

2. Open `http://app.fizzy.localhost:3006`.

3. Log in with `david@example.com`. The development page shows the verification code.

4. Stop the server with `Ctrl-C`.

Fizzy uses SQLite for development Action Cable. It does not need Redis or a separate database server.

## Lobsters

The `lobsters/lobsters` configuration uses Ruby `4.0.0` and libvips. It includes `libyaml` and `pkg-config` to build native gems.

The setup task installs gems, copies `config/database.yml.sample` if `config/database.yml` does not exist, and prepares the databases. Rails adds seed data when it creates the databases.

1. Run these commands from the Lobsters directory:

   ```zsh
   devenv-allow lobsters/lobsters
   devenv tasks run lobsters:setup
   devenv shell bin/dev
   ```

2. Open `http://localhost:3000`.

3. Log in with username `test` and password `test`.

4. Stop the server with `Ctrl-C`.

Lobsters uses SQLite for its databases, cache, and job queue. It does not need Redis or a separate database server.

The `sqlite3` gem builds SQLite with the options in `.bundle/config`. Use `devenv shell bin/sqlite3` instead of the system SQLite client.
