# tracklist-to-apple-music

Turn any plain-text tracklist into a real Apple Music playlist, with every song checked against Apple's catalogue first.

AI chat tools are good at writing playlists and bad at delivering them. You get 100 lines of `Artist - Title`, a few of them misspelt or invented, and an afternoon of searching and tapping "Add to Playlist". This repo does the whole job in two commands' worth of effort:

1. `resolve.py` matches each line to an exact song in the Apple Music catalogue and tells you which lines are solid, which need a second look, and which do not exist in your country's store.
2. `build_playlist.js`, generated for you by step 1, creates the playlist in your library with the tracks in order. It syncs to your phone, Mac and everything else signed in to your Apple Account.

The example in this repo is a 130-track retro-espionage lounge playlist that started as a Gemini conversation: Digby Jones, Kinobe, Air, Portishead, John Barry and friends.

## Quick start

You need Python 3.9 or later (standard library only), an Apple Music subscription, and a desktop browser.

```bash
git clone https://github.com/RADobson/tracklist-to-apple-music.git
cd tracklist-to-apple-music

python3 resolve.py playlists/operation-sun-drenched-rendezvous.txt \
  --country AU \
  --name "Operation: Sun-Drenched Rendezvous" \
  --description "130 tracks of retro-espionage lounge"
```

That writes five files to `out/`:

| File | What it is |
|---|---|
| `report.md` | Every line with the song it matched, the album, and a confidence score |
| `build_playlist.js` | The playlist builder, with your song IDs already filled in |
| `links.txt` | One Apple Music link per matched song, in playlist order |
| `ids.txt` | The Apple Music song IDs, in playlist order |
| `matched.json` | Full detail, also used as a cache so re-runs are instant |

Read `report.md`, fix any lines you want to change in your tracklist, and run the command again. Only the changed lines are looked up.

Then build the playlist:

1. Open [music.apple.com](https://music.apple.com) in a desktop browser and sign in.
2. Open the JavaScript console. In Chrome that is View > Developer > JavaScript Console.
3. Paste the contents of `out/build_playlist.js` and press Return. Chrome asks you to type `allow pasting` the first time.

The console prints `Created "<name>" with <n> tracks`, and the playlist appears in your library within a few seconds.

## Tracklist format

One track per line. The album is optional and steers the match towards a particular release.

```text
# Lines starting with # are comments
Digby Jones - Pina Colada (Jazz Mix)
Air - All I Need | Moon Safari
Portishead - Cowboys | Portishead
```

Numbered and bulleted lists work too, so you can paste straight from a chat window. To read from the clipboard on a Mac:

```bash
pbpaste | python3 resolve.py - --name "My Playlist"
```

## How matching works

For each line the resolver asks Apple's public iTunes Search API for up to 50 candidates and scores them.

- **Identity** is a 0 to 1 score built from artist and title similarity. It ignores accents, punctuation and case, understands guest credits, and marks karaoke and tribute versions down hard. A score of 0.86 or higher counts as confident, 0.70 to 0.86 goes to the review list, and anything lower is reported as missed.
- **Release ranking** then picks between several releases of the same song. It favours the album you named, streamable releases, and the plain studio recording ahead of live cuts, remixes and compilations.

Apple allows roughly 20 lookups a minute on this API, so the resolver paces itself. Allow about six minutes per 100 tracks on the first run.

Useful options:

| Option | Meaning |
|---|---|
| `--country` | Storefront to match against, such as `AU`, `US` or `GB`. Catalogues differ by country. |
| `--name`, `--description` | Playlist name and description |
| `--include-review` | Also add the "review" matches to the playlist |
| `--confident`, `--review` | Change the score thresholds |
| `--out` | Output folder |

## What the example run found

The 130-track list came out of an AI chat. Against the Australian store:

- 125 lines matched exactly as written.
- 2 titles were slightly wrong: "Vai Vem" is "Vai Vai", and Montefiori Cocktail's "Huahine" is "Hu Ha".
- 3 tracks are missing from the Australian catalogue: the Supreme Beings of Leisure debut album and two cuts from Kruder & Dorfmeister's G-Stoned EP. The example list swaps in other tracks by the same artists.
- Several songs first matched a live cut, a remix or an unstreamable soundtrack release. Album hints fixed those.

The full result is in [`examples/operation-sun-drenched-rendezvous/report.md`](examples/operation-sun-drenched-rendezvous/report.md).

## Good to know

- **The builder is a short, readable script. Read it before you paste it.** That is good practice for anything you run in a browser console. It makes two kinds of request: one to create the playlist and one per 25 tracks to fill it. It checks that Apple returned the new playlist by name before adding anything.
- **The builder runs inside your own signed-in tab** and talks to Apple Music through the web player. Apple can change the web player whenever it likes. If the builder stops working, please open an issue. Tested with Chrome on macOS in November 2026.
- **Why the browser?** On a Mac, AppleScript and Shortcuts reach the songs already in your library. Catalogue songs you have yet to add are out of their reach, and Apple's official API needs a paid developer account. The web player covers that gap for free.
- **Your `links.txt` works anywhere.** Each link opens the exact matched song, so you can also hand the file to a playlist transfer tool or share it with a friend.

## Licence

MIT. See [LICENSE](LICENSE).
