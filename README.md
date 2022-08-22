# Config Management

Manage config across multiple workstations using [chezmoi](https://www.chezmoi.io)

# Configure new machine

Before running `chezmoi` on MacOS install xcode command line tools.

```
$ xcode-select --install
$ sh -c "$(curl -fsLS chezmoi.io/get)"
```

Then setup variables that chezmoi will need for rendering templates. In this setup there are few params for gitconfig and gpg key id, this need to be generated or imported before running `chezmoi apply`
github ssh keys need to be setup before running `chezmoi init` command. [Generate new](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) or import from another location before proceeding.
`chezmoi init` requires all values used in the .gitconfig template to be set. One of them is gpg key for commits signing which does not exist on a clean machine before necessary tools are installed. It can be set to a temp value and later edited manually.

```bash
$ chezmoi edit-config # see below, all values need to be filled before running next command
$ chezmoi init git@github.com:<USER>/dotfiles.git
$ chezmoi apply -v
```

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

