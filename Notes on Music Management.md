I used [Beets](https://beets.readthedocs.io/en/stable/index.html) to reorganize my music and get out of the clutches of the 'Music' App which is a buggy, unergonomic, unmaintained, embarrassing piece of shit by a company that has a market cap higher than oil companies.

I have about ~24,000 tracks and spent a cumulative of a week importing them into Beets. It uses the lovely and amazing SQLite for its database, and not some bullshit proprietary format, making it possible to write [all sorts of UIs](https://github.com/afreeorange/rosolli) on top of it ❤️

Did all of this on macOS 13.2.1 (Ventura)

## Working with Beets

Just random notes and not a comprehensive guide. The documentation is great.

### Basic Configuration

```yaml
directory: ~/Music/Library
library: ~/Music/Library/library.db
ignore:
    - '*.flac'

import:
    move: yes

plugins:
    - convert
    - discogs
    - duplicates
    - edit
    - embedart
    - fetchart
    - fromfilename
    - fuzzy
    - info
    - lastgenre
    - lyrics
    - mbsync
    - missing
    - play
    - web

play:
    command: /opt/homebrew/bin/VLC

convert:
    command: ffmpeg -i $source -ab 320k -ac 2 -ar 48000 -map_metadata 0 -id3v2_version 3 $dest
    extension: mp3

discogs:
    user_token: iHoejCCcOBhqOKZPnFTZlJCcyhZzLyaaryPiOinZ
    index_tracks: yes

edit:
    itemfields:
        - artist
        - album
        - album_id
        - track
        - title
        - genre
```

- Discogs was _extremely_ helpful, especially with Indian stuff and radio shows.
- Set `EDITOR="subl -w"` before running `beet edit foo` since [multiple cursors](https://vimeo.com/401631075?embedded=true&source=vimeo_logo&owner=27762735) save a lot of time when editing tag data.
- [The Beets FAQ](https://beets.readthedocs.io/en/latest/faq.html) is great

### Searching and Listing Things

```bash
# List things with formatting
beet ls -f '$id - $track/$tracktotal - $album - $title' Massive Attack 100th

# List the exact album
# https://github.com/beetbox/beets/issues/4371
beet ls artist:Pink Floyd album:~'The Wall'
```

### Database Schema

`beet fields` will give you [a list of all the fields/columns](https://beets.readthedocs.io/en/latest/reference/pathformat.html#available-values) you'll find the database. [You can add your own too](https://beets.io/blog/flexattr.html)! I used the excellent [`litecli`](https://litecli.com/) for a lot of explorations.

## Editing Metadata

I use the awesome [mp3Tag](https://www.mp3tag.de/en/). If on Linux, `beet edit` with your favorite editor will work just fine (except for album art... not sure how you'd do that at the CLI).

## Syncing with my Phone

Created a playlist in the Music App called "Things to Sync" and just synced that to my phone. I tried to get out of Apple's clutches by using [iMazing](https://imazing.com/) but it didn't work: it would stall on the last song to sync and _appeared_ to corrupt the music database. Just stuck with the Finder integration.

## Syncing from Beets &rarr; Apple Music

I largely gave up on doing this automatically and sync changes manually. It sucks. But that's how it is. See the "References" section for more notes and frustrations.

In Apple Music &rarr; Preferences &rarr; Files, uncheck both

- Keep Music Media Folder organized (lol)
- Copy files to Music Media Folder

You can keep the location of the Music Media Folder wherever you'd like. Here's my layout:

```
/Users/nikhil
  ├── Library     <- This is my Beets library
  └── Music       <- Apple's Database
```

Now for Create/Update/Delete operations:

- When I Create/Delete an album, I manually Create/Delete it in the Music app.
- The Music app is not bad as a general metadata editor. So when I Update track(s) using the editor, I run `beet update` to sync changes to the Beets database.

This has worked well so far. It's manual and against every instinct I have as a programmer but my stuff doesn't change too often for me to worry about learning AppleScript (which may be dead soon because Apple).

## Dealing with FLAC Files

I converted these to 320Kbps MP3 for maximum comatibility. [X Lossless Decoder](https://tmkk.undo.jp/xld/index_e.html) is an OG converter for macOS but I ended up using the venerable `ffmpeg`.

```bash
# Convert FLAC to 320kbps MP3
# I kept the FLAC files around too...
for i in *.flac
do
    ffmpeg  -i "$i" \
            -ab 320k \
            -map_metadata 0 \
            -id3v2_version 3 \
            "`basename "$i" .flac`.mp3"
done
```

To split FLAC files, I used [xACT](http://xact.scottcbrown.org/) will split the FLAC files based on CUE files for macOS. It uses `shntool` for this. So if you do this at the command-line,

```bash
# Split FLAC files according to CUE files (macOS)
brew install cuetools flac ffmpeg shntool
shnsplit -o flac -f file.cue file.flac

# Use cuetag to tag the newly split files based on information from the CUE files
# You get this from cuetools but here's a bash script:
# https://github.com/gumayunov/split-cue/blob/master/cuetag
cuetag.sh file.cue split-track*.flac
```

## Issues with Beets

I'd run `beet update` and get this weirdness

```
The Beatles - The Capitol Albums, Vol. 1 - If I Fell (mono)
  albumtype: compilation -> a
  albumtypes: album; compilation -> ['a', 'l', 'b', 'u', 'm', ';', ' ', 'c', 'o', 'm', 'p', 'i', 'l', 'a', 't', 'i', 'o', 'n']
```

Was able to 'fix' this temporarily based on [this issue](https://laxmicontestgame.com/?_=%2Fbeetbox%2Fbeets%2Fpull%2F4582%23MEFDg0OUwunSMl5Y8erm91wq#issuecomment-1445023493):

```bash
beet mbsync 'albumtypes::^\['
```

## References and Notes

### The Frustrating State of AppleScript

You can script things in macOS in AppleScript or JavaScript. The [documentation](https://developer.apple.com/library/archive/documentation/LanguagesUtilities/Conceptual/MacAutomationScriptingGuide/index.html#//apple_ref/doc/uid/TP40016239-CH56-SW1) is old and pretty garbage and you'll have to rely on Google and SO to do things.

If, like me, you're excited that you can write stuff in JavaScript, calm your horses. There's no code-completion, no types, and no IDE support in XCode since they removed the AppleScript template in v14 (you have to copy it over from v13). This has led to at least one person on the internet asking if Apple will deprecate AppleScript. [And a petition](https://forum.latenightsw.com/t/petition-apple-for-applescript-3-0/3897/6). [And this salty-ass reply from a mature dev](https://forum.latenightsw.com/t/petition-apple-for-applescript-3-0/3897/6) exploring the history of AppleScript.

It's all rather shitty. Avoid using the Music app as much as possible and use iMazing to transfer stuff to your iPhone until that company sadly announces that Apple has blocked them from doing their thing as well.

- [Apple's dumb MusicDB format](https://fileinfo.com/extension/musicdb)
- [Remove Dead Tracks](https://www.leawo.org/entips/remove-dead-tracks-in-itunes-1114.html) (based on [this script](https://dougscripts.com/itunes/scripts/ss.php?sp=mxremovedeadsuper) that's available for $1.99)
- AppleScript References: [One](https://apple.stackexchange.com/questions/449991/remove-russian-songs-from-itunes-with-applescript), [Two](https://256stuff.com/gray/docs/misc/itunes_applescript_commands/), [Three](https://gist.github.com/nerdEd/547738), [Four](https://gist.github.com/nerdEd/547738), [Five](https://gist.github.com/ccstone/716173d330caa118d09c)
- [A Collection of macOS Automation Sites](http://macosxautomation.com/)
- [Scripting Terminology in AppleScript](https://developer.apple.com/library/archive/documentation/LanguagesUtilities/Conceptual/MacAutomationScriptingGuide/AboutScriptingTerminology.html#//apple_ref/doc/uid/TP40016239-CH75-SW2)

Here are some cached AppleScripts...

### Add Tracks

This is AppleScript. You run this [as a hook](https://beets.readthedocs.io/en/stable/plugins/hook.html) upon adding a song. The argument is the absolute path to the song. Re-running this won't add the song twice.

```applescript

-- Reference: https://randomgeekery.org/post/2017/10/beets-and-itunes/

on run (argv)
    tell application "Music"
        set filename to (argv's item 1 as string)
        try
            set trackRef to filename
            refresh trackRef
        end try
    end tell
end run
```

### Refreshing the Library

Especially when you delete a song in `beets` and want that reflected in your Music app. This is slow but whatever.

```applescript
-- Reference:
-- https://www.leawo.org/entips/remove-dead-tracks-in-itunes-1114.html

tell application "Music"
    set mainLibrary to library playlist 1
    set totalTracks to count of file tracks of mainLibrary
    set deleted_tracks to 0

    log "There are " & totalTracks & " tracks in the library."
    log "Scanning... this will take a while."

    repeat with t from totalTracks to 1 by -1
        set the_track to file track t of mainLibrary

        if the_track's location is missing value then
            log "MISSING: " & the_track
            delete the_track
            set deleted_tracks to deleted_tracks + 1
        end if
    end repeat

    log "I deleted " & deleted_tracks & " tracks."
end tell
```

### Remove Dead Tracks

```applescript

-- remove dead files
-- Reference: https://gist.github.com/aleemb/97b4f5f8510f397c4a08

property progress_factor : 500

tell application "Music"
    display dialog "Super Remove Dead Tracks" & return & return & Â
        "Removes tracks whose files are missing or deleted, so-called" & Â
        "\"dead\" tracks designated with (!) next to the track name." buttons Â
        {"Cancel", "Proceed..."} default button 2

    set mainLibrary to library playlist 1
    set totalTracks to count of file tracks of mainLibrary
    set all_playlists to user playlists whose smart is false -- don't delete Smart Playlists later

    set deleted_tracks to 0
    set all_checked_tracks to 0
    set countem to ""

    set oldfi to fixed indexing
    set fixed indexing to true

    repeat with t from totalTracks to 1 by -1

        try
            set this_track to file track t of mainLibrary
            -- set _duration to duration of this_track
            -- set _title to name of this_track

            set loc to "." & this_track's location

            if loc is missing value or loc contains ".Trash" or _title is missing value then
                display dialog "Deleting: " & _title as string
                delete this_track
                set deleted_tracks to deleted_tracks + 1
            end if

            set all_checked_tracks to all_checked_tracks + 1

            if frontmost then
                if (progress_factor is not 0) and (all_checked_tracks mod progress_factor) is 0 then
                    if deleted_tracks is greater than 0 then Â
                        set countem to (deleted_tracks & " dead tracks removed so far...")
                    if frontmost is true then display dialog (all_checked_tracks as string) & Â
                        " tracks checked..." & Â
                        return & countem buttons {"Cancel", Çdata utxt266BÈ} giving up after 1
                end if
            end if
        end try

    end repeat

    set fixed indexing to oldfi

    repeat with this_playlist in all_playlists
        if (get count of tracks of this_playlist) is 0 then
            try
                delete playlist this_playlist
            end try
        end if
    end repeat

    if deleted_tracks is greater than 0 then
        set ps to " was"
        if deleted_tracks is not 1 then set ps to "s were"
        display dialog "Removed " & deleted_tracks & " dead track" & ps & Â
            "." buttons {"Thanks"} default button 1 with icon 1
    else
        -- if gave up of (display dialog "No dead tracks found." buttons {"Thanks"} Â
        --  default button 1 with icon 1 giving up after 15) is true then error number -128
    end if

end tell
```
