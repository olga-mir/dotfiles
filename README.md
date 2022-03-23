# Config Management

Manage config across multiple workstations using [chezmoi](https://www.chezmoi.io/quick-start/#start-using-chezmoi-on-your-current-machine)

Vim config managed in separate repo [dotvim](https://github.com/olga-mir/dotvim)

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

```bash
chezmoi add  -T --autotemplate ~/.gitconfig
```

This will create `dot_gitconfig.tmpl` file in `chezmoi` folder.
