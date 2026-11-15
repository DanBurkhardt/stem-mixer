# Stem Mixer

A single-file web app for practicing along with song stems. Open a folder of stems, and every audio file becomes a track with its own volume, pan, mute and solo. Mute your own part and play along.

It was built for a cover band that splits songs with StemRoller and wanted something quicker than loading every song into GarageBand or Logic.

- No server, no build step, no dependencies. The whole app is `stem-mixer.html`.
- Audio never leaves your computer. The browser reads the files from your disk and plays them from memory.

## Quick start

1. Open `stem-mixer.html` in a browser (double-click it, or host it anywhere that serves static files).
2. Press **Open stem folder** and pick a song's folder.
3. Mute your part and press Play.

The browser's file dialog may call this an "upload". Nothing is uploaded.

## Files

| File | Purpose |
| --- | --- |
| `stem-mixer.html` | The app. Markup, styles, script and the guide graphics are all in this one file. |
| `favicon.ico` | Site icon, 16 to 256 px. Put it next to the page when hosting. |
| `favicon.gif` | Static 32×32 version of the same icon. |

## Stem folders

The app expects one folder per song, with files named the way StemRoller writes them:

```
Artist - Song Title/
  Artist - Song Title - vocals.wav
  Artist - Song Title - drums.wav
  Artist - Song Title - bass.wav
  Artist - Song Title - other.wav
  Artist - Song Title - instrumental.wav
  Artist - Song Title - original.wav
```

- The track name is the last ` - ` part of the file name, in capitals (`VOCALS`, `DRUMS`, and so on). The rest becomes the song title.
- `instrumental` and `original` are full mixes, so they start muted.
- Accepted file types: `.wav`, `.mp3`, `.flac`, `.m4a`, `.aac`, `.ogg`, `.opus`, `.aif`, `.aiff`, as far as the browser can decode them.
- If you pick a parent folder that holds several song folders, a song picker appears in the top bar.
- Very large songs trigger a prompt that offers to skip the full mixes, because decoded audio takes far more memory than the files take on disk.

## Features

**Tracks**
- Volume fader, pan knob, mute and solo per track, plus a master fader.
- Click a track name to rename it (up to 28 characters). Clear the name to restore the original. The audio files are never renamed.
- Zoomable waveforms for every track.

**Transport**
- Play / pause, rewind, fast-forward, back to start.
- Play range: drag along the bar above the tracks to mark a section. Double-click the range to clear it.
- Repeat: loops the play range, or the whole song if no range is set.

**Click track and count-in**
- The tempo and beat positions are detected from the audio when a song loads, and a generated `CLICK` track is added. It starts muted.
- Corrections for when the detection is off: `÷2` and `×2` (half or double tempo), `½ SHIFT` (click lands on the off-beat), nudge in milliseconds, beats per bar, and `SET BAR 1`.
- Count-in: 1 to 8 clicks before the music starts, or off. Default is 4. It plays when you press Play, not when you jump around or when a loop restarts.
- If no steady beat is found, the click track and count-in are turned off for that song.

**Saved settings**
- **Save settings** stores the current song's mix: volume, pan, mute, solo, track names, play range, repeat and tempo corrections.
- When you open the same song again, the app asks whether to use the saved settings or the defaults.
- Settings live in a browser cookie when the page is hosted, and in `localStorage` when it is opened as a local file. They are tied to that browser on that computer.

**Built-in help**
- **Guide**: an annotated picture of the interface with ten numbered tips.
- **How it works**: how a playlist becomes stems, with a Basic view and a More technical view.

## Keyboard

| Key | Action |
| --- | --- |
| `Space` | Play / pause |
| `←` `→` | Skip 5 seconds |
| `Home` | Back to the start (of the play range, if one is set) |
| `R` | Repeat on / off |
| `I` `O` | Set the play range start / end at the playhead |
| `+` `-` | Zoom in / out |
| `Ctrl` or `⌘` + scroll | Zoom at the pointer |
| `Esc` | Close the Guide or How it works |

Also: click the timeline to jump, and double-click a fader or knob to reset it.

## How the stems are made

The app only plays stems. This is the workflow it was built around:

1. **Playlist.** A Spotify playlist of the songs to learn.
2. **Links.** Claude finds the studio version of each track on YouTube and lists the URLs.
3. **Audio.** `yt-dlp` saves each one as an MP3:
   ```
   yt-dlp -x --audio-format mp3 --audio-quality 0 "<watch URL>"
   ```
4. **Stems.** StemRoller splits each MP3 into separate instrument files, one folder per song.
5. **Practice.** Open a song's folder in Stem Mixer.

One person can do steps 1 to 4 for the whole band. Everyone else only needs the stem folders and this page.

Use audio you have the right to use. Downloading and sharing copyrighted recordings can breach a service's terms or copyright law, depending on where you are.

## Hosting

Copy `stem-mixer.html` and `favicon.ico` to any static host. Rename the page to `index.html` if you want it at the site root.

The host only ever serves the page. Visitors' audio files are read locally in their own browser and are not sent to the host.

## How it works inside

- **Playback:** Web Audio API. Each stem is decoded into an `AudioBuffer` and played through its own gain and stereo panner into a master gain. All stems start at the same audio-clock time, and loop passes are scheduled a few seconds ahead so loops stay tight.
- **Waveforms:** minimum and maximum peaks are precomputed at several resolutions and drawn on a canvas, only for the visible part of the timeline.
- **Beat detection:** an onset envelope, a tempo estimate that is allowed to drift over the song, then dynamic-programming beat tracking. Clicks are scheduled on the audio clock along that beat grid.
- **Settings:** stored under a key made from a hash of the song's file names, so the same folder is recognized next time.

## Status and limits

- Tested in headless Chrome with synthetic test stems. Not yet tested in Safari or Firefox, or tuned on real recordings.
- Beat detection can report half or double the real tempo. Use `÷2` / `×2` to fix it.
- Odd meters may lock to a plain quarter-note pulse.
- Stems are decoded fully into memory, so many long songs' worth of tracks need a lot of RAM. Closing the tab frees it.
- Not packaged as a Mac or Windows app yet. It is plain web files, so it can be wrapped later.
