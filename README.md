# [**HideOut**](https://ivorypawn.github.io/)

[Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod/) + Github Pages.

The following blog post was a great help in layouts and other overrides:
*[Overview of Hugo/PaperMod, modifying PaperMod, and comparison to al-folio](https://jessewei.dev/blog/2023/papermod/)*

## commands

Initialize the theme after cloning:

```sh
git submodule update --init --recursive
```

Use Hugo 0.138.0 (the version used to generate this site). PaperMod requires
Hugo 0.125.7 or newer. Restart the server after installing the theme or changing
the Hugo executable.

```sh
hugo new notes/Test/index.md
hugo new blog/yyyy-mm-dd/index.md

hugo
hugo server
```

If using the project-local Windows executable:

```powershell
.\.tools\hugo\hugo.exe
.\.tools\hugo\hugo.exe server --renderToMemory
```

If `hugo version` still reports an older Chocolatey installation, update it
from an administrator PowerShell:

```powershell
choco upgrade hugo --version=0.138.0 --yes
hugo version
```
