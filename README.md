# mpv-anime-skip

Mpv script for skipping anime openings and endings. Uses anitopy, shikimori's api and api.aniskip.com.

## How it works

On keypress, the script parses the currently playing filename, resolves it to a MyAnimeList ID via Shikimori, and asks AniSkip for opening/ending timestamps for that exact episode. If the current playback position falls inside one of those ranges, it seeks to the end of it.

## Features

- OP/ED skip on a single keybind
- Fuzzy title matching with explicit season handling — including anime split into "Part 1"/"Part 2" MAL entries, and fansub groups that number episodes straight through multiple seasons without resetting
- Works with network streams too (e.g. ani-cli piping HLS into mpv), not just local files
- Cross-platform: Linux, macOS, Windows
- A uosc menu: submitting missing timestamps (staged from mpv's own A-B loop), voting on existing ones, re-checking AniSkip on demand, and jumping straight to the title's Shikimori/MAL page
- Per-episode local cache — no repeated network requests on every keypress

## Dependencies

- [curl](https://curl.se/) — ships by default on Windows 10 1803+, macOS, and most Linux distros
- [Python 3](https://www.python.org/)
- [anitopy](https://github.com/igorcmoura/anitopy) — filename parsing. **Not installed via pip** in this setup; the `anitopy` package folder from that repo is vendored directly inside this project (see the layout below). anitopy is licensed under MPL-2.0 — its license file ships inside the vendored `anitopy/` folder and covers only that folder, not the rest of this project.
- [uosc](https://github.com/tomasklaen/uosc) — optional, only needed for the menu (submit/vote/refresh/links). The automatic skip keybind works without it.

## Installation

1. Copy this project files into `scripts/anime-skip/`.
2. Copy `anitopy` folder from [here](https://github.com/igorcmoura/anitopy) into `scripts/anime-skip/`.
2. Make sure `curl` and `python3` are available in `PATH`.
3. If your system's Python binary is called `python` instead of `python3`, change `PYTHON_CMD` at the top of `main.lua`.
4. If you placed the folder somewhere other than `scripts/anime-skip/`, update `ANITOPY_LOOKUP_PATH` and `MATCH_SCRIPT_PATH` accordingly.

It must be something like this:
```
~/.config/mpv
├── cache
├── fonts
├── input.conf
├── mpv.conf
├── script-opts
├── scripts
│   ├── anime-skip
│   │   ├── anitopy
│   │   ├── lookup.py
│   │   ├── main.lua
│   │   └── match.py
└───└── uosc
```

## Keybindings

The script does **not** bind any keys by default. Add your own to `input.conf`:

```
F1 script-binding anime-skip         # perform the skip
F2 script-binding anime-skip-menu    # open the submit/vote/links menu (requires uosc)
```

## The menu

Opening the menu shows what was resolved for the current file — title, season/episode, MAL ID, and either the found timestamps or the reason nothing was found — followed by whatever's relevant right now:

- **Open on Shikimori / Open on MAL** — jump to the title's page
- **Refresh from AniSkip** — re-check AniSkip for this exact episode without re-parsing the filename or re-searching Shikimori (shown when AniSkip is missing at least one segment)
- **Stage current loop as Opening/Ending** + **Submit** — mark a region with mpv's own A-B loop (`l` twice) and submit it to AniSkip (shown per-segment, only for segments AniSkip doesn't have yet)
- **Rate timestamp correct/wrong** — vote on existing timestamps (shown for segments AniSkip already has, except ones you submitted yourself)

## Known limitations

- Filename parsing depends on anitopy; unusual naming schemes can still fail
- Season/part matching is heuristic (fuzzy title matching + explicit season-marker detection); very obscure or inconsistently-named titles can be mismatched or wrongly rejected
- Depends on Shikimori and AniSkip being reachable; both are free community services and can go down or change domains without notice

## Troubleshooting

- **Suddenly nothing resolves anymore:** Shikimori has moved domains before (`.one` → `.io`, `.one` is blocked in some regions). Check `SHIKIMORI_GRAPHQL_URL` in `main.lua` still points somewhere reachable.
- **A file is stuck showing an old/wrong result:** the per-episode cache lives in `~~/cache/anime-skip_cache/` (i.e. `~/.config/mpv/cache/anime-skip_cache/`). Delete the relevant file (or the whole folder) to force a re-check.
- **A request failed with no useful detail:** failed responses that aren't valid JSON get dumped to `~~/cache/anime-skip_cache/last_error_*.txt` for inspection.

## Credits

- [anitopy](https://github.com/igorcmoura/anitopy) (MPL-2.0) — filename parsing
- [Shikimori](https://shikimori.io) — title search and MAL ID resolution
- [AniSkip](https://api.aniskip.com) — skip timestamp data
- Originally inspired by [synacktraa/ani-skip](https://github.com/synacktraa/ani-skip)
