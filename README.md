# Config Management

Manage config across multiple workstations using [chezmoi](https://www.chezmoi.io)

# Configure new machine

## Prerequisites

**Step 1: Xcode command line tools** — required before anything else. This installs Apple's developer tools (git, clang, etc.) which Homebrew depends on. This does NOT install Homebrew itself.

```
$ xcode-select --install
```

**Step 2: Brew**

Download `pkg` file from latest release and install: https://github.com/Homebrew/brew/releases

**Step 3: SSH keys** — needed to clone the dotfiles repo via git. [Generate new](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) or import from another machine before proceeding.

**Step 4: Install chezmoi**

```
$ sh -c "$(curl -fsLS get.chezmoi.io)"
```

## Apply dotfiles

Set up variables chezmoi needs for rendering templates (git name/email/GPG signing key). The GPG key doesn't exist yet on a clean machine — use a placeholder and update it later.

```bash
$ chezmoi edit-config # fill in name, email, signingkey before running next command
$ chezmoi init git@github.com:<USER>/dotfiles.git
$ chezmoi apply -v
```

`chezmoi apply` will automatically:
- Install Homebrew if not present (the install script bootstraps it)
- Add Homebrew to `$PATH` if needed (handles Apple Silicon's `/opt/homebrew` prefix)
- Install all packages via `brew bundle`

# Template existing config

Using variables for personal information or settings that are different between workstations.

Check what `chezmoi` can gather:

```bash
chezmoi data
```

These values can be automatically templated, but additional data can be supplied if needed by using:
```bash
chezmoi edit-config
```

and add required values, e.g. for .gitconfig:
```toml
[data]
         name = "value"
         email = "value"
         signingkey = "temp-value"
```
Note that values must be quoted.

Use the following command to generate template (this is not needed on the new machine or when using existing template):
```bash
chezmoi add  -T --autotemplate ~/.gitconfig
```

This will create `dot_gitconfig.tmpl` file in `chezmoi` folder.

# Not covered

Currently not covered by install script:

* Visual Studio Code
  use "Settings Sync" extention to sync complete setup on another machine.
  the plugin did not port all setting smoothly, manual fixes: https://github.com/VSCodeVim/Vim#mac
* Docker
* SSH key management (needed before chezmoi init to pull repo)
* GPG key management
* Vim config managed in separate repo [dotvim](https://github.com/olga-mir/dotvim)

