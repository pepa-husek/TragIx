# Zpěvníky oddílů Ichthys

TragIx = Tragoudi Ixtys = Zpívající Ryba

## Zpěvníky ke stažení

- [Nezbožnej (červený) · aktuální PDF](https://github.com/pepa-husek/TragIx/releases/latest/download/nezboznej.pdf)
- [Zbožnej (zelený) · aktuální PDF](https://github.com/pepa-husek/TragIx/releases/latest/download/zboznej.pdf)
- [Archiv vydání](https://github.com/pepa-husek/TragIx/releases)

Scout troop songbook build system. Two variants:

- **Nezbožnej** (červený/red) -- secular campfire songs
- **Zbožnej** (zelený/green) -- gospel/church songs

## Repository structure

```
common/                Shared TeX
nezboznej/             Red songbook variant
  head.tex             Title page and preamble
  tail.tex             Chord diagrams and closing
  songs/               Individual song files (canonical, clean UTF-8)
  overrides/           Build-time layout overrides (spacing hacks etc.)
  build/               Build output (generated, gitignored)
zboznej/               Green songbook variant (same structure)
fonts/                 Custom Scout fonts (TheMix C5, SKAUT)
scripts/               Helper scripts
doc/                   Documentation and notes
zdroje/                Legacy song source archives
build_songbook.py      Main build script
current_editions.txt   Which variant+edition+version combos to build
Dockerfile             Reproducible build environment
```

## Building

```bash
# Build everything listed in current_editions.txt:
python build_songbook.py --all

# Or build a specific variant+edition:
python build_songbook.py --variant nezboznej --edition 2025
python build_songbook.py --variant zboznej --edition 2026
```

If `luatex` is not installed locally the script automatically builds
and runs inside Docker -- no wrapper scripts needed. Just have Docker
running and the above command works on both Windows and Linux/Mac.

The `--edition` parameter selects songs whose `E:` header includes
that year (e.g. `E: 2025` or `E: 2007, 2012, 2025`). Overrides from
`overrides/` are used when present.

The `current_editions.txt` file at the repo root controls which
variant+edition+version triples are active. The CI workflow and `--all`
flag both read from it. To change what gets built, edit that one file.
The version number appears on the title page and in the output PDF
filename (e.g. `nezboznej_2025.3.pdf`). Bump it when you rebuild.

When publishing a GitHub release as **Latest**, attach both current PDFs as
`nezboznej.pdf` and `zboznej.pdf`. Include the current PDF for the other variant
when only one has changed, and record both edition versions in the release notes.

Options:

```bash
--all          Build all editions from current_editions.txt
--version N    Preview a version of the selected edition
--duplex       Also produce an A5-on-A4 duplex PDF for the print shop
--clean        Only clean the build directory, then exit
--songs-only   Collect and write song files but do not compile the PDF
```

### Building without Docker

Install the TeX toolchain locally and the script will use it directly:

```bash
sudo apt install -y python3 texlive texlive-luatex texlive-plain-generic \
  texlive-lang-czechslovak texlive-fonts-recommended poppler-utils psutils ghostscript
sudo locale-gen cs_CZ.UTF-8
```

## Contributing songs

Songs are plain text files -- editable with any text editor, including
GitHub's built-in editor. No special tooling required.

### Editing a song

1. Navigate to `nezboznej/songs/` or `zboznej/songs/` on GitHub
2. Open the song file (e.g. `Přítel.txt`)
3. Click the pencil icon to edit
4. Make your changes and click **Propose changes**
5. This creates a Pull Request that the maintainer can review and merge

### Adding a new song

1. Click **Add file** in the appropriate `songs/` directory
2. Paste the song content in the [standard format](#song-file-format)
3. Make sure the `E:` header field includes the current edition year
4. For Nezbožnej 2025, also list the filename in the
   [supplement manifest](nezboznej/editions/2025.toml).

### Removing a song from an edition

Edit the song file and remove the edition year from the `E:` field.
The song file stays in `songs/` for potential future use.

## Git conventions

Write commit subjects as `type(scope): short action`, for example
`fix(chords): resolve diagram coverage warnings`. Use `feat` for features,
`fix` for corrections, `docs` for documentation, and `chore` for maintenance.
Keep each commit focused on one change that can be reviewed on its own.

Use `<prefix>/<short-kebab-case-topic>` for branches. Examples include
`feat/nezboznej-supplements`, `fix/chord-diagram-coverage`, and
`codex/nezboznej-2025-supplements`. Use these conventions for new work and
preserve published history so collaborators can pull normally.

## Song file format

Full specification: `doc/song_format.txt` (English, technical).
Contributor-friendly version: `doc/jak_psat_pisnicky.txt` (Czech).

Quick reference:

- Header: `N:` name, `A:` author, `AC:` author code, `Z:` source, `ZC:` source code, `E:` editions, `K:` liturgy category
- Verses: `1:`, `2:`, ... Refrains: `R:`, `R1:`, ... Bridge: `M:` Recitative: `C:`
- Chords inline before syllables: `(G)`, `(Am7)`, `(C"m)` (sharp = `"`, flat = `b`)
- Continuation lines: indented with spaces, or prefixed with `+`
- Repetition: `/: ... :/` or `3×/: ... :/`
- Comments: lines starting with `%`

## Helper scripts

```bash
# Quality check -- single file or whole directory
python scripts/check_songbook.py nezboznej/songs/Přítel.txt
python scripts/check_songbook.py nezboznej/songs/ --tail nezboznej/tail.tex

# Only check songs in a specific edition
python scripts/check_songbook.py nezboznej/songs/ --edition 2025

# Auto-fix safe issues (preview first, then apply)
python scripts/check_songbook.py nezboznej/songs/ --fix --dry-run
python scripts/check_songbook.py nezboznej/songs/ --fix
```

## Editions

Active build targets are listed in `current_editions.txt`.

| Variant         | 2007 | 2012 | 2025        | 2026        |
| --------------- | ---- | ---- | ----------- | ----------- |
| Nezbožnej (red) | yes  | yes  | **current** | --          |
| Zbožnej (green) | yes  | yes  | --          | in progress |


## Nezbožnej 2025 supplements

The [original page list](nezboznej/editions/2025.pages) fixes the order and
printed page numbers of the 228 original songs. Keep supplement songs in
`nezboznej/songs/` with `E: 2025`. Add their filenames to
the applicable `[[supplements]]` block in
the [edition manifest](nezboznej/editions/2025.toml). The build sorts each
supplement by title in Czech alphabetical order. Set `heading` to the text
printed above the first song, for example `Dodatek z listopadu 2026`.

Preview an update with `python build_songbook.py --variant nezboznej --edition 2025 --version 4`.
Once the layout is approved, record the printed start pages in the batch's
`pages` list, paired with the filenames in its `songs` list, and update
`current_editions.txt`. The build checks those page numbers on subsequent
runs. A later batch gets a new block and the next release version that
includes it.

## Overrides

The `overrides/` directory holds build-time layout modifications for
already-printed editions (invisible spacing, placeholder songs, etc.).
During build, an override file takes precedence over the matching `songs/`
file. New editions start without overrides; they are created only during
final layout fine-tuning before print.

## Known issues (Nezbožnej)

These are intentionally left unfixed in the current printed edition:

- "Sbohem Galánečko", "Marnivá sestřenice", "Kozel", "Tři myši prašivý"  
have non-standard repetition marks or have them missing.
- "Mršina" has inconsistent line breaks across verses due to long lines  
that don't fit properly.

## Future ideas

- Capo as a header field (currently written as free text in the song).
- Time signature (takt) as a header field.
- Triple-dot semantics: currently `...` means both "sing the obvious"
and "fade out" -- consider differentiating these.
- Most header fields (AC, Z, ZC) are underused -- reconsider whether
they should stay required.

