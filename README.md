# Config Management

Manage config across multiple workstations using [chezmoi](https://www.chezmoi.io)

Vim config managed in separate repo [dotvim](https://github.com/olga-mir/dotvim)

# Configure new machine

Before running `chezmoi` on MacOS install xcode command line tools.

```
$ xcode-select --install
$ sh -c "$(curl -fsLS chezmoi.io/get)"
```

Then setup variables that chezmoi will need for rendering templates. In this setup there are few params for gitconfig and gpg key id, this need to be generated or imported before running `chezmoi apply`
```
$ chezmoi edit-config # see below, all values need to be filled before running next command
$ chezmoi init https://github.com/username/dotfiles.git
$ chezmoi apply -v
```

## Template

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
```
Note that values must be quoted.

Use the following command to generate template (this is not needed on the new machine or when using existing template):
```bash
chezmoi add  -T --autotemplate ~/.gitconfig
```

This will create `dot_gitconfig.tmpl` file in `chezmoi` folder.
