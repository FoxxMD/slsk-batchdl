# slsk-batchdl

A batch downloader for Soulseek built with Soulseek.NET. Accepts CSV files and Spotify or YouTube urls.

## Examples

Download tracks from a csv file:
```
sldl test.csv
```
<details>
  <summary>CSV details</summary>
  
The names of the columns in the csv should be: `Artist`, `Title`, `Album`, `Length`. Some alternatives are also accepted. You can use `--print tracks` before downloading to check if everything has been parsed correctly. Only the title or album column is required, but additional info may improve search results.  

</details> 

<br>

Download spotify likes while skipping songs that already exist in the output folder:
```
sldl spotify-likes --skip-existing
```
<details>
  <summary>Spotify details</summary>

To download private playlists or liked songs you will need to provide a client id and secret, which you can get here https://developer.spotify.com/dashboard/applications. Create an app and add `http://localhost:48721/callback` as a redirect url in its settings.  

</details> 

<br>

Download from a youtube playlist with fallback to yt-dlp in case it is not found on soulseek, and retrieve deleted video titles from wayback machine:
```
sldl "https://www.youtube.com/playlist?list=PLI_eFW8NAFzYAXZ5DrU6E6mQ_XfhaLBUX" --get-deleted --yt-dlp
```
<details>
  <summary>YouTube details</summary>

Playlists are retrieved using the YoutubeExplode library which unfortunately doesn't always return all videos. You can use the official API by providing a key with `--youtube-key`. Get it here https://console.cloud.google.com. Create a new project, click "Enable Api" and search for "youtube data", then follow the prompts.
Also note that due the high number of music videos in the above example playlist, it may be better to remove all text in parentheses and disable song duration checking: `--regex "[\[\(].*?[\]\)]" --pref-length-tol -1`.

</details> 

<br>

Search & download a specific song, preferring lossless:
```
sldl "title=MC MENTAL @ HIS BEST,length=242" --pref-format "flac,wav"
```  
<details>
  <summary>String details</summary>

  The shorthand `sldl "Artist - Title"` is equivalent to `sldl "artist=Artist,title=Title"`.

</details> 

<br>

Interactive album download:
```
sldl "album=Some Album" --interactive
```
<details>
  <summary>Album details</summary>

  The shorthand `sldl "Artist - Album" -a` is equivalent to `sldl "artist=Artist,album=Album"`. It's often helpful to restrict to folders which have two or more tracks: `--atc 2+`.

</details> 
  
<br>

Print all songs by an artist which are not in your library:
```
sldl "artist=MC MENTAL" --aggregate --skip-existing --music-dir "path/to/music" --print tracks-full
```

## Download Modes

Depending on the provided input, the download behaviour changes:

- Normal download: The program will download a single file for every input entry.
- Album download: The program will search for the album and download an entire folder including non-audio files. Activated when the input is a link to a spotify or bandcamp album, when the input string or csv row has no track title, or when `-a`/`--album` is enabled.
- Aggregate download: With `-g`/`--aggregate`, the program will first perform an ordinary search for the input, then attempt to group the results into distinct songs and download one of each kind. This can be used to download an artist's entire discography (or simply printing it, like in the example above), finding remixes of a song, etc. Note that it is not 100% reliable, which is why `--min-users-aggregate` is set to 2 by default, i.e. any song that is shared by only one person will be ignored. Enable `--relax-filtering` to make the file filtering less aggressive.

## Options

Acronyms of two- and --three-word-arguments are also accepted, e.g. --twa
```
Usage: sldl <input> [OPTIONS]

  <input>                        <input> is one of the following:

                                 Spotify playlist url or 'spotify-likes': Download a spotify
                                 playlist or your liked songs. --spotify-id and
                                 --spotify-secret may be required in addition.

                                 Youtube playlist url: Download songs from a youtube playlist.
                                 Provide a --youtube-key to include unavailabe uploads.

                                 Path to a local CSV file: Use a csv file containing track
                                 info to download. The names of the columns should be Artist,
                                 Title, Album, Length. Only the title or album column is
                                 required, but extra info may improve search results.

                                 Name of the track, album, or artist to search for:
                                 Can either be any typical search string or a comma-separated
                                 list like 'title=Song Name,artist=Artist Name,length=215'
                                 Allowed properties are: title, artist, album, length (sec)
                                 Specify artist and album only to download an album.

Options:
  --user <username>              Soulseek username
  --pass <password>              Soulseek password

  -p --path <path>               Download directory
  -f --folder <name>             Subfolder name. Set to '.' to output directly to the
                                 download folder (default: playlist/csv name)
  -n --number <maxtracks>        Download the first n tracks of a playlist
  -o --offset <offset>           Skip a specified number of tracks
  -r --reverse                   Download tracks in reverse order
  --name-format <format>         Name format for downloaded tracks, e.g "{artist} - {title}"
  --fast-search                  Begin downloading as soon as a file satisfying the preferred
                                 conditions is found. Higher chance to download wrong files.
  --remove-from-source           Remove downloaded tracks from source playlist or CSV file
                                 (spotify and CSV only)
  --m3u <option>                 Create an m3u8 playlist file
                                 'none': Do not create a playlist file
                                 'fails' (default): Write only failed downloads to the m3u
                                 'all': Write successes + fails as comments

  --spotify-id <id>              spotify client ID
  --spotify-secret <secret>      spotify client secret

  --youtube-key <key>            Youtube data API key
  --get-deleted                  Attempt to retrieve titles of deleted videos from wayback
                                 machine. Requires yt-dlp.
  --deleted-only                 Only retrieve & download deleted music. Combine with --print
                                 tracks-full to display a list of all deleted titles & urls.

  --time-format <format>         Time format in Length column of the csv file (e.g h:m:s.ms
                                 for durations like 1:04:35.123). Default: s
  --yt-parse                     Enable if the csv file contains YouTube video titles and
                                 channel names; attempt to parse them into title and artist
                                 names.

  --format <format>              Accepted file format(s), comma-separated
  --length-tol <sec>             Length tolerance in seconds
  --min-bitrate <rate>           Minimum file bitrate
  --max-bitrate <rate>           Maximum file bitrate
  --min-samplerate <rate>        Minimum file sample rate
  --max-samplerate <rate>        Maximum file sample rate
  --min-bitdepth <depth>         Minimum bit depth
  --max-bitdepth <depth>         Maximum bit depth
  --banned-users <list>          Comma-separated list of users to ignore

  --pref-format <format>         Preferred file format(s), comma-separated (default: mp3)
  --pref-length-tol <sec>        Preferred length tolerance in seconds (default: 3)
  --pref-min-bitrate <rate>      Preferred minimum bitrate (default: 200)
  --pref-max-bitrate <rate>      Preferred maximum bitrate (default: 2500)
  --pref-min-samplerate <rate>   Preferred minimum sample rate
  --pref-max-samplerate <rate>   Preferred maximum sample rate (default: 48000)
  --pref-min-bitdepth <depth>    Preferred minimum bit depth
  --pref-max-bitdepth <depth>    Preferred maximum bit depth
  --pref-banned-users <list>     Comma-separated list of users to deprioritize

  --strict-conditions            Skip files with missing properties instead of accepting by
                                 default; if --min-bitrate is set, ignores any files with
                                 unknown bitrate.

  -a --album                     Album download mode
  -t --interactive               When downloading albums: Allows to select the wanted album
  --album-track-count <num>      Specify the exact number of tracks in the album. Folders
                                 with a different number of tracks will be ignored. Append
                                 a '+' or '-' after the number for the inequalities >= and <=
  --album-ignore-fails           When downloading an album and one of the files fails, do not
                                 skip to the next source and do not delete all successfully
                                 downloaded files
  --album-art <option>           Retrieve additional images after downloading the album:
                                 'largest': Download from the folder with the largest image
                                 'most': Download from the folder containing the most images
  --album-art-only               Only download album art for the provided album

  -g --aggregate                 Instead of downloading a single track matching the input,
                                 find and download all distinct songs associated with the
                                 provided artist, album, or track title.
  --min-users-aggregate <num>    Minimum number of users sharing a track before it is
                                 downloaded in aggregate mode. Setting it to higher values
                                 will significantly reduce false positives, but also cause it
                                 to ignore rarer songs. Default: 2
  --relax-filtering              Slightly relax file filtering in aggregate mode to include
                                 more results

  -s --skip-existing             Skip if a track matching file conditions is found in the
                                 output folder or your music library (if provided)
  --skip-mode <mode>             'name': Use only filenames to check if a track exists
                                 'name-precise' (default): Use filenames and check conditions
                                 'tag': Use file tags (slower)
                                 'tag-precise': Use file tags and check file conditions
                                 'm3u': Skip all tracks that don't have a fail entry in m3u
  --music-dir <path>             Specify to also skip downloading tracks found in a music
                                 library. Use with --skip-existing
  --skip-not-found               Skip searching for tracks that weren't found on Soulseek
                                 during the last run. Fails are read from the m3u file.

  --no-remove-special-chars      Do not remove special characters before searching
  --remove-ft                    Remove 'feat.' and everything after before searching
  --remove-brackets              Remove square brackets and their contents before searching
  --regex <regex>                Remove a regexp from all track titles and artist names.
                                 Optionally specify a replacement regex after a semicolon.
                                 Add 'T:', 'A:' or 'L:' at the start to only apply this to
                                 the track title, artist, or album respectively.
  --artist-maybe-wrong           Performs an additional search without the artist name.
                                 Useful for sources like SoundCloud where the "artist"
                                 could just be an uploader. Note that when downloading a
                                 YouTube playlist via url, this option is set automatically
                                 on a per-track basis, so it is best kept off in that case.
  -d --desperate                 Tries harder to find the desired track by searching for the
                                 artist/album/title only, then filtering. (slower search)
  --yt-dlp                       Use yt-dlp to download tracks that weren't found on
                                 Soulseek. yt-dlp must be available from the command line.
  --yt-dlp-argument <str>        The command line arguments when running yt-dlp. Default:
                                 "{id}" -f bestaudio/best -cix -o "{savepath}.%(ext)s"
                                 Available vars are: {id}, {savedir}, {savepath} (w/o ext).
                                 Note that with -x, yt-dlp will download webms in case
                                 ffmpeg is unavailable.

  -c --config <path>             Set config file location
  --search-timeout <ms>          Max search time in ms (default: 5000)
  --max-stale-time <ms>          Max download time without progress in ms (default: 50000)
  --concurrent-downloads <num>   Max concurrent downloads (default: 2)
  --searches-per-time <num>      Max searches per time interval. Higher values may cause
                                 30-minute bans. (default: 34)
  --searches-renew-time <sec>    Controls how often available searches are replenished.
                                 Lower values may cause 30-minute bans. (default: 220)
  --display-mode <option>        Changes how searches and downloads are displayed:
                                 'single' (default): Show transfer state and percentage
                                 'double': Transfer state and a large progress bar
                                 'simple': No download bars or changing percentages
  --listen-port <port>           Port for incoming connections (default: 50000)

  --print <option>               Print tracks or search results instead of downloading:
                                 'tracks': Print all tracks to be downloaded
                                 'tracks-full': Print extended information about all tracks
                                 'results': Print search results satisfying file conditions
                                 'results-full': Print search results including full paths
  --debug                        Print extra debug info
```

### File conditions
Files not satisfying the conditions will not be downloaded. Files satisfying `pref-` conditions will be preferred; setting `--pref-format "flac,wav"` will make it download high quality files if they exist, and only download low quality files if there's nothing else. Conditions can also be supplied as a semicolon-delimited string to `--cond` and `--pref`, e.g `--cond "br>=320;f=mp3,ogg;sr<96000"`. See the start of `Program.cs` for the default file conditions.

**Important note**: Some info may be unavailable depending on the client used by the peer. For example, the default Soulseek client does not share the file bitrate. By default, if `--min-bitrate` is set, then files with unknown bitrate will still be downloaded. You can configure it to reject all files where one of the checked properties is unavailable by enabling `--strict-conditions`. (As a consequence, if `--min-bitrate` is also set then any files shared by users with the default client will be ignored)

### Name format
Variables enclosed in {} will be replaced by the corresponding file tag value. Available variables are: artist, sartist, artists, albumartist, albumartists, title, stitle, album, salbum, year, track, disc, filename, foldername. The variables sartist, stitle and salbum will be replaced by the source artist, title and album respectively (i.e what is shown in spotify/youtube/csv file) instead of the tag values of the downloaded file. Name format supports subdirectories as well as conditional expressions like `{tag1|tag2}` – If tag1 is null, choose tag2. String literals enclosed in parentheses are ignored in the null check. Examples:
- `{artist} - {title}`: Always name it 'Artist - Title'. Because some files on Soulseek do not have tags, the second example is preferred:
- `{artist( - )title|filename}`: If artist and title is not null, name it 'Artist - Title', otherwise use the original filename.

### Quality vs Speed
The following options will make it go faster, but may decrease search result quality or cause instability:
- `--fast-search` skips waiting until the search completes and downloads as soon as a file matching the preferred conditions is found
- `--concurrent-downloads` -  set it to 4 or more
- `--max-stale-time` is set to 50 seconds by default, so it will wait a long time before giving up on a file
- `--searches-per-time` increase at the risk of ban, see the notes section for details.

### Quality vs Quantity
The options `--strict-title`, `--strict-artist` and `--strict-album` will filter any file that does not contain the title/artist/album in the filename (ignoring case, bounded by boundary chars). Another way to prevent false downloads is to set `--length-tol` to 3 or less to make it ignore any songs that differ from the input by more than 3 seconds. However, all 4 options are already enabled as 'preferred' conditions by default, meaning that such files will only be downloaded as a last resort even without enabling these. Hence it is only recommended to enable them if you need to minimize false downloads as much as possible.

## Configuration  
Create a file named `sldl.conf` in the same directory as the executable and write your arguments there, e.g:
```
username = fakename
password = fakepass
pref-format = flac
fast-search = true
```  
  
## Notes
- For macOS builds you can use publish.sh to build the app. Download dotnet from https://dotnet.microsoft.com/en-us/download/dotnet/6.0, then run `chmod +x publish.sh && sh publish.sh`. For intel macs, uncomment the x64 and comment the arm64 section in publish.sh. 
- `--display single` and especially `double` can cause the printed lines to be duplicated or overwritten on some configurations. Use `simple` if that's an issue.
- The server will ban you for 30 minutes if too many searches are performed within a short timespan. The program has a search limiter which can be adjusted with `--searches-per-time` and `--searches-renew-time` (when limit is reached, the status of the downloads will be "Waiting"). By default it is configured to allow up to 34 searches every 220 seconds. These values were determined through experimentation as unfortunately I couldn't find any information regarding soulseek's rate limits, so they may be incorrect.

## Docker

A docker container for running `sldl` can be built from this repository. The image supports linux x86/ARM. 

To build and start container:

```shell
clone https://github.com/fiso64/slsk-batchdl
cd slsk-batchdl
docker compose up -d
```

`exec` into the container to start using `sldl`:

```shell
docker compose exec sldl sh
sldl --help
```

The compose stack mounts two directories relative to where `docker-compose.yml` is located which can be used for file management:

* `/config` (at `./config` on host) - put your `sldl.conf` [configuration](#configuration-) in this directory and then use `sldl -c /config ...` to use your configuration in the container
* `/data` (at `./data` on host) - use as the download directory IE `sldl -p /data ...`

### File Permissions

If you are running Docker on a **Linux Host** you should specify `user:group` permissions of the user who owns the **configuration and data directory** on the host to avoid [docker file permission problems.](https://ikriv.com/blog/?p=4698) These can be specified using the [environmental variables **PUID** and **PGID**.](https://docs.linuxserver.io/general/understanding-puid-and-pgid)

To get the UID and GID for the current user run these commands from a terminal:

* `id -u` -- prints UID
* `id -g` -- prints GID

Replace these with the corresponding variable (`PUID` `PGID`) in `docker-compose.yml`.


### Cron

One or more `sldl` commands can be run on a schedule using [cron](https://en.wikipedia.org/wiki/Cron) built into the container.

To create a schedule make a new file on the host `./config/crontabs/abc` and use it with the standard [crontab](https://en.wikipedia.org/wiki/Cron#Overview) syntax.

Make sure to restart the container after any changes to the cron file are made.

Example => Run `sldl` every Sunday at 1am, search for all tracks from the specified Spotify playlist

```
# min   hour    day     month   weekday command
0 1 * * 0 sldl https://open.spotify.com/playlist/6sf1WR5grXGJ6dET -c /config -p /data"
```

[crontab.guru](https://crontab.guru/) could be used to help with the scheduling expression.