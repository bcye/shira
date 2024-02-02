# Glomatico's YouTube Music Downloader (KraXen's Fork)
Download YouTube Music songs/albums/playlists with tags from YouTube Music in 256kbps AAC/128kbps Opus/128kbps AAC.
This fork adds several new features as well as many song tagging improvements.

## New Features
- Downloading from Soundcloud supported
- Downloading of non-music youtube videos as music supported
- Added more cli options & flags
- planned/wip
  - WebUI based on WebSockers (WIP/alpha)
  - GUI to crop thumbnail for non-music videos
  - GUI to confirm tags for non-music videos

## Tagging improvements
- uses video's `upload_date` for more precise release date when possible
- tries to resolve MusicBrainz ID's from their api 
  - `track`, `album`, `artist`, `albumartist` ids, falls back to `artist`, `albumartist` (sometimes artist existst in MusicBrainz DB but song doesen't)
- cleans up messy titles into more reasonable ones:
  - `IDOL【ENGLISH EDM COVER】「アイドル」 by YOASOBI【Aries Shepard x @djJoMusicChannel 】` =>
  - `IDOL [ENGLISH EDM COVER] [アイドル] by YOASOBI`
- uses my custom smart-metadata system from [tiger](https://github.com/KraXen72/tiger) for non-music videos
- uses a smart algorithm to determine if video's thumbnail should be cropped or padded to 1:1 aspect ratio
  
<details id="smartcrop">
<summary>More info about cropping algorithm</summary>
<ul>
<li>samples 4 pixels near the corners of the thumbnail (smoothed and reduced to 64 colors).</li>
<li>decides to crop if average of standard deviations of r, g and b color channels from each sample point is lower than a than a treshold</li>
<li>otherwise pads the image to 1:1 with it's dominant color</li>
</ul>
</details>


## Why not just use yt-dlp directly?
While this project uses yt-dlp under the hood, it has the advantage of utilizing [YouTube Music's API](https://github.com/sigma67/ytmusicapi) to get songs metadata. This includes information such as track number, square cover, lyrics, year, etc.

## Setup (development)
1. clone git repo
2. have `python` and `ffmpeg` installed, as mentioned below
3. `pip install -r requirements.txt`
4. `python -m gytmdl <link>` to run

## Setup
> currently, this fork is not published on pip
1. Install Python 3.8 or higher (3.11+ recommended)
2. Install gytmdl with pip
    ```
    pip install gytmdl
    ```
3. Add FFmpeg to PATH or specify the location using the command line arguments or the [config file](#configuration)
   - to easily install ffmpeg on windows, you can install [scoop](https://scoop.sh) and run `scoop install main/ffmpeg`
4. (optional) Set a cookies file
   * By setting a cookies file, you can download age restricted tracks, private playlists and songs in 256kbps AAC if you are a premium user. You can export your cookies to a file by using the following Google Chrome extension on YouTube Music website: https://chrome.google.com/webstore/detail/gdocmgbfkjnnpapoeobnolbbkoibbcif.

## Usage examples
Download a song:
```
gytmdl "https://music.youtube.com/watch?v=3BFTio5296w"
```
Download an album:
```
gytmdl "https://music.youtube.com/playlist?list=OLAK5uy_lvpL_Gr_aVEq-LaivwJaSK5EbFd4HeamM"
```

## Configuration
gytmdl can be configured using the command line arguments or the config file. The config file is created automatically when you run gytmdl for the first time at `~/.gytmdl/config.json` on Linux and `%USERPROFILE%\.gytmdl\config.json` on Windows. Config file values can be overridden using command line arguments.
| Command line argument / Config file key | Description | Default value |
| --- | --- | --- |
| `-f`, `--final-path` / `final_path` | Path where the downloaded files will be saved. | `./YouTube Music` |
| `-t`, `--temp-path` / `temp_path` | Path where the temporary files will be saved. | `./temp` |
| `-c`, `--cookies-location` / `cookies_location` | Location of the cookies file. | `null` |
| `--ffmpeg-location` / `ffmpeg_location` | Location of the FFmpeg binary. | `ffmpeg` |
| `--config-location` / - | Location of the config file. | `<home folder>/.gytmdl/config.json` |
| `-i`, `--itag` / `itag` | Itag (audio quality). | `140` |
| `--cover-size` / `cover_size` | Size of the cover.  [0<=x<=16383] | `1200` |
| `--cover-format` / `cover_format` | Format of the cover. [jpg\|png] | `jpg` |
| `--cover-quality` / `cover_quality` | JPEG quality of the cover.  [1<=x<=100] | `94` |
| `--cover-img` / `cover_img` | Path to image or folder of images named video/song id  | `null` |
| `--cover-crop` / `cover_crop` |  'crop' takes a 1:1 square from the center, pad always pads top & bottom [auto\|crop\|pad] | `auto` - [more](#smartcrop) |
| `--template-folder` / `template_folder` | Template of the album folders as a format string. | `{album_artist}/{album}` |
| `--template-file` / `template_file` | Template of the song files as a format string. | `{track:02d} {title}` |
| `-e`, `--exclude-tags` / `exclude_tags` | List of tags to exclude from file tagging separated by commas without spaces. | `null` |
| `--truncate` / `truncate` | Maximum length of the file/folder names. | `40` |
| `-l`, `--log-level` / `log_level` | Log level. | `INFO` |
| `-s`, `--save-cover` / `save_cover` | Save cover as a separate file. | `false` |
| `-o`, `--overwrite` / `overwrite` | Overwrite existing files. | `false` |
| `-p`, `--print-exceptions` / `print_exceptions` | Print exceptions. | `false` |
| `-u`, `--url-txt` / - | Read URLs as location of text files containing URLs. | `false` |
| `-n`, `--no-config-file` / - | Don't use the config file. | `false` |

### Itags
The following itags are available:
- `140` (128kbps AAC) - default, because it's the result of `bestaudio/best` on a free account
- `141` (256kbps AAC) - use if you have premium alongside `--cookies location`
- `251` (128kbps Opus) - most stuff will error with `Failed to check URL 1/1`. Better to use `140`
  
SoundCloud will always download in 128kbps MP3
> SoundCloud also offers OPUS, however, [some people were complaining](https://www.factmag.com/2018/01/04/soundcloud-mp3-opus-format-sound-quality-change-64-128-kbps/) that the quality is worse  
> [These are questionable claims](https://old.reddit.com/r/Techno/comments/bzodax/soundcloud_compression_128kbps_mp3_vs_64_kbps/) at best, but better safe than sorry.  
> SoundCloud OPUS support might come later.  

### Cover formats
Can be either `jpg` or `png`.

### Cover img
- If you pass in a path to an image file, it will get used for all of the songs you're currently downloading.
- If you pass in a path to a folder, the script will look in the folder and use the first image matching the song/video id and of jpeg/png format as a cover
  - You don't have to create covers for all songs in the playlist/album/etc. you're downloading.
  - SoundCloud will also consider images based on the URL slug
    - for example: `https://soundcloud.com/yatashi-gang-63564467/lovely-bastards-yatashigang` => `lovely-bastards-yatashigang.jpg` or `.png`

### Tag variables
The following variables can be used in the template folder/file and/or in the `exclude_tags` list:

- `title`
- `album`
- `artist`
- `album_artist`
- `track`
- `track_total`
- `release_year`
- `release_date`
- `cover`
- `comment`
- `lyrics`
- `media_type`
- `rating`
- `track`
- `track_total`
