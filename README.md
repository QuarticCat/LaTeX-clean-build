# LaTeX Clean Build

When building latex documents, there will be a lot of temporary files like `*.aux`, `*.log`, `*.toc`, etc. Apart from these commonly seen files, many packages will create their own temporary files for caching or other purposes. For example, `minted` will create a directory starts with `_minted-`, and `tikz` with `\tikzexternalize` will create PDF files for each graph. Then what if you don't want to see them?

Usually we cannot simply delete temporary files by some regular expression rules. Since that will probably slow down the next build, and these rules are hard to share between different documents.

Some LaTeX engines provide an option to designate another output directory, like `-output-directory` in XeLaTeX. However, many packages cannot work properly when the output directory is not current directory. So this option only works in limited cases.

There is another solution: use [tectonic](https://github.com/tectonic-typesetting/tectonic). This is an excellent project and I really admire their works. But it comes with a totally different design concept. And you have to accept all of them. Say, if you only want a clean build but not separate packages, you have no way to do this. Besides, currently it is not perfectly compatible with most documents. If you use `tikz` with `\tikzexternalize`, tectonic will go wrong.

Here I provide another way to do a LaTeX clean build, which might meet your needs. The code is in [build-latex.zsh](/build-latex.zsh). The idea of this script is:

1. Create a temporary directory in a cache directory (usually `~/.cache` or `$XDG_CACHE_HOME`).
1. Copy all files to this directory. To avoid copy cost, we can instead create links.
1. Set the working directory to this directory, and the build the document in the original directory. This step is to make sure all temporary files are created in temporary directory while keep the path in `*.syntex.gz` pointing to original TeX files so that SyncTeX can work as expected.
1. Copy needed files back to the original directory. And also, you can create links.

This script is just an example. Please feel free to modify it or write your own one. The important thing in this repo is the idea.

## Usage

```console
$ ./build-latex.zsh <latex-engine-command> <doc-without-file-extension>
```

E.g.

```console
$ ./build-latex.zsh 'xelatex -synctex=1 -interaction=nonstopmode -file-line-error -shell-escape' 'Report'
```

It is easy to work with [LaTeX Workshop](https://github.com/James-Yu/LaTeX-Workshop). Here is a config example:

```json
{
    "latex-workshop.latex.tools": [
        {
            "name": "xelatex",
            "command": "build-latex.zsh",
            "args": [
                "xelatex -synctex=1 -interaction=nonstopmode -file-line-error -shell-escape",
                "%DOCFILE%",
            ],
        },
        {
            "name": "bibtex",
            "command": "build-latex.zsh",
            "args": [
                "bibtex",
                "%DOCFILE%",
            ],
        },
    ],
    "latex-workshop.latex.recipes": [
        {
            "name": "xe",
            "tools": [
                "xelatex",
            ],
        },
        {
            "name": "xe->xe",
            "tools": [
                "xelatex",
                "xelatex",
            ],
        },
        {
            "name": "xe->bib->xe->xe",
            "tools": [
                "xelatex",
                "bibtex",
                "xelatex",
                "xelatex",
            ],
        },
    ],
}
```

And of course you should make sure `build-latex.zsh` is in the PATH exported to VS Code, otherwise you should write the full path in `command` field.
