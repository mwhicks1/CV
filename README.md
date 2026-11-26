# Publications bibliography

`mwh.bib` is the master list of my publications. The scripts in `html/` turn it
into the publication pages of my website (https://mhicks.me), using a
customized `bibtex2html`.

## Layout

| Path | What it is |
|---|---|
| `mwh.bib` | The bibliography |
| `mwh.tex` | Small LaTeX file for checking that `mwh.bib` runs through BibTeX |
| `html/updatebib` | Runs the whole pipeline (see below) |
| `html/genbib`, `year2bib`, `submitbib` | Build the by-year page, `mwh.html` |
| `html/gencatbib`, `category2bib`, `*.desc` | Build the by-category page, `mwh-cat.html`; each `.desc` is the blurb for one category |
| `html/rmback` | Strips "Back" links from generated pages |
| `html/html2md` | Turns the generated `.html` pages into the website's `content/papers/*.md` |
| `html/title-plain.bst` | BibTeX style used for the pages |
| `bibtex2html/` | Submodule: upstream [bibtex2html](https://github.com/backtracking/bibtex2html), pinned to one commit |
| `patches/bibtex2html-mwh.patch` | My customizations to bibtex2html (adds `-nometafields`, changes the per-paper page layout) |
| `patches/bibtex2html-configure` | Upstream's generated `configure` script, which isn't in their repo |
| `build-bibtex2html` | Builds the patched `bibtex2html` and `bib2bib` and installs them to `~/bin` |

Generated files (`html/*.html`, `html/tmp/`, `build/`, LaTeX outputs) are not
tracked.

## Adding a paper

1. Add the entry to `mwh.bib`. Give it a `category` matching one of the
   categories listed in `html/gencatbib`. Entries are listed by `year`;
   `@techreport` entries (used for arXiv-only preprints, with
   `institution = {CoRR}`, `number = {abs/<id>}`) go in each year's
   "Technical Reports" subsection. `omitfromweb` hides an entry; a `submitted`
   field or a note containing "submit" lists it under submitted papers.
2. `cd html && ./updatebib`
3. In the website repo, check `git status`: new and updated pages are in
   `content/papers/`. Preview with `hugo server`, then commit there.

`html2md` never deletes pages. If an entry is removed from or renamed in
`mwh.bib`, it lists the leftover page as `stale:`; delete it from the
website's `content/papers/` by hand.

`updatebib` lists years from 1997 through next year. A new category needs
its name added to the list in `html/gencatbib` and a `html/<Category>.desc`
blurb.

## Building bibtex2html

    git submodule update --init
    ./build-bibtex2html

Needs OCaml, `make` and `patch` (not autoconf, and not hevea; the manual is
skipped). The previously installed binaries are kept as `~/bin/*.prev`. Use
`--no-install` to build without installing, or set `BINDIR` to install
elsewhere.

## Upgrading bibtex2html

1. Move the submodule to the new upstream commit:

       git -C bibtex2html fetch && git -C bibtex2html checkout <commit>

2. Rebase the patch onto it in a scratch clone and regenerate
   `patches/bibtex2html-mwh.patch` with `git format-patch -1 --stdout`.
   If upstream changed `configure.in`, regenerate
   `patches/bibtex2html-configure` with `autoconf`.
3. `./build-bibtex2html --no-install`
4. Compare against the current version: copy `mwh.bib` and the `html/` scripts
   into two scratch directories, run `gencatbib ../mwh.bib`,
   `genbib ../mwh.bib 1997 <year>` and `rmback` in each, one with the old
   binaries first on `PATH` and one with `build/bibtex2html/` first, then
   `diff -r` the two outputs.
5. If the differences are acceptable, run `./build-bibtex2html` to install,
   and commit the submodule and patch together.

The current pin (upstream `c138eb2`, Jan 2023, still version 1.99) was compared
this way against the previous build (upstream `f1b6b57`): the only differences
were that links to `https` URLs are now labelled "https" instead of "http",
and an `https` URL in an `opturl` field is now a clickable link.
