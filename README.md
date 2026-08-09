# Scoring with Lidarr
*V2026.07.11-01*
## A path to success using naming convention, custom formats, quality profile and delay profile

*This guide is not a "Lidarr A-Z", I can only recommend you starting with **[Servarr Wiki](https://wiki.servarr.com/en/lidarr)** to gain an in-depth understanding of how it works.
If you're already familiar with it, this guide is intended to help you get the best out of its automatic download system.*

## Preamble

I've never been satisfied with [Davo's guide on Servarr's wiki](https://wiki.servarr.com/en/lidarr/community-guide), I sometimes got multiple downloads of the same release of the same quality when the cutoff is unmet (multiple MP3 320 before the FLAC, FLAC24 followed by a FLAC some minutes after), so I wanted to dig a bit more into it and adapt what I learned from my experience with Profilarr to Lidarr.

The biggest flaw in Davo's guide, in my opinion, is the naming convention. The scoring system they suggest is decent during the search stage, but once an album is downloaded, imported into your library and then renamed, the files won't have enough information to be scored as they should (hence the multiple downloads of the same release, at least partially). The Custom formats, as they are, can only be useful for scoring search result, not the files and folders in the torrent, nor their renamed versions.

## How the scoring system works in Lidarr

For this demonstration, I'll use a simple profile based on Davo's guide.

![Screenshot 1](https://i.ibb.co/LzX7P5Bn/Screenshot-2026-07-11-192154.png)

There are 3 stages where Lidarr scores a release:

## 1 - The announce name

It can be from the RSS feeds, an automatic or a manual search.

Here is a search result:

![Screenshot 2](https://i.ibb.co/zVdJHsVk/Screenshot-2026-07-11-202211.png)

Let's say only MP3 releases are available at the moment, so this one is grabbed (automatically with the announce or by search).

## 2 - The torrent folders and files names

Once grabbed, the torrent will appear on your Activity page.

![Screenshot 3](https://i.ibb.co/Ng6qFL5x/Screenshot-2026-07-11-202507.png)

It's not the case in this example, but its score can then be really different, regardless of the profile you're using. The reason is that the scoring system now tests against the folder name in the torrent, not the release name anymore.

And when importing the complete torrent, the tracks are scored too, but only using the file name. So there's a good chance of having no score at all, depending on the naming convention used by the uploader.

![Screenshot 4](https://i.ibb.co/jkLj4dyX/Screenshot-2026-07-11-204951.png)

## 3 - The imported and renamed files

It's the most important part to avoid duplicate downloads. And with this quality profile and naming convention, nothing is scored here.

For example, if I launch a new search, nothing prevents Lidarr from downloading another torrent with the same or a lower score than the one I already have, it will just prevent me from downgrading the **Quality**.

![Screenshot 5](https://i.ibb.co/ksMNVgJH/Screenshot-2026-07-11-205152.png)

No reject logo is present next to the download button.

So if a new MP3 torrent is announced on a tracker, it will be automatically downloaded.

Only once a FLAC version has been imported, and the cutoff is met, will the multiple auto-downloads finally stop.

## How to do better
***Please note the following:**
-  I don't use hardlinks, as I let Lidarr keep the metadata in sync with MusicBrainz. Hardlinks can help a lot with space, you're encouraged to use it. Just remember: **DO NOT** use hardlinks AND change the metadata!
-  I'm using Jellyfin as a media player. The naming convention I propose can and should be adapted to your preferences and your media player. Take time to test each part jumping from Lidarr to your refreshed library.
-  My quality goal is FLAC 16bit coming from digital sources, so the custom formats and scoring used in this guide should be adapted to your liking.*

## Naming convention

My settings for the naming:

> <u>**Rename Tracks**</u>: V

> <u>**Replace Illegal Characters**</u>: V

> <u>**Colon Replacement**</u>: Smart Replace

> <u>**Standard Track Format/Multi Disc Track Format**</u>: {Release Year} - {Album CleanTitle} {(Album Disambiguation)} {[Album Type]} {[Album MbId]}/{medium:00} - {Medium Name} [{Medium Format}] - {track:00} - {Track ArtistCleanName} - {Track CleanTitle} [{Quality Title}]

> <u>**Artist Folder Format**</u>: {Artist CleanNameThe} {[Artist Disambiguation]}

About the Track Format:
- I use the same format for both standard and multi disc for consistency. Also, because Jellyfin sees each disc as a different release if I separate the discs by folder, I don't use subfolders.
- The naming I suggest for the tracks folder contains the year and clean title of the release group, its disambiguation if specified, and its format and MBID. Adapt to your preferences, but I think it's the better way to keep clarity and avoid issues with identically named album/single/EP released the same year. The disambiguation is really optional, rarely used. On the contrary the MBID is the most important part for this purpose.
- Once again, you can adapt my suggestion for the track naming to your preferences, but please note that the name of each track **MUST** contain the **{Medium Format}** and **{Quality Title}** tokens, as it will allow Lidarr to correctly score the release once imported. You can take examples on [those conventions](https://wiki.servarr.com/lidarr/naming-guide#community-naming-conventions), but none of them use the **{Medium Format}** token in the track name, only on the subfolder.

## Custom formats

### FLAC

This one is for FLAC 16bit only, my quality goal in this guide.

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "FLAC",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "FLAC",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "^(?=.*(?i:\\bFLAC\\b))(?!.*(?i:(?:\\b24(?:[- ]?bit)?\\b|\\b24[-/]\\d{2,3}\\b))).*$"
      }
    }
  ]
}
```

</details>

### Lossless

This one is for all other lossless files, negated by the previous non-24bit FLAC regex.

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "Lossless",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "lossless",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": "^(?:(?=.*(?i:\\b(?:lossless|alac|wavpack|wv|ape|wave)\\b))|(?=.*(?i:\\bFLAC\\b))(?=.*(?i:(?:\\b24(?:[- ]?bit)?\\b|\\b24[-/]\\d{2,3}\\b)))).*$"
      }
    },
    {
      "name": "FLAC: FLAC",
      "implementation": "ReleaseTitleSpecification",
      "negate": true,
      "required": true,
      "fields": {
        "value": "^(?=.*(?i:\\bFLAC\\b))(?!.*(?i:(?:\\b24(?:[- ]?bit)?\\b|\\b24[-/]\\d{2,3}\\b))).*$"
      }
    }
  ]
}
```

</details>

### MP3 TOP

This one is for MP3 320 and MP3 V0 (VBR).

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "MP3 Top",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "MP3 Top",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "^(?=.*(?i:\\bMP3\\b))(?=.*(?:\\b320\\b|\\bV0(?:[- ]?VBR)?\\b)).*$"
      }
    }
  ]
}
```

</details>

### Lossy

This one is for all other lossy files.

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "Lossy",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Lossy",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "^(?:(?=.*(?i:\\bMP3\\b))(?!.*(?:\\b320\\b|\\bV0(?:[- ]?VBR)?\\b))|(?=.*\\bOGG\\b)|(?=.*\\bAAC\\b)|(?=.*\\bWMA\\b)).*$"
      }
    }
  ]
}
```

</details>

### CD

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "CD",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "CD",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "\\bCD\\b"
      }
    }
  ]
}
```

</details>

### WEB

About this one: I've added to this regex the formats used by MusicBrainz for those releases. Some may be missing, tell me if you know some to add.

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "WEB",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "WEB",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "\\bWEB|Digital Media|Download Card|USB Flash Drive\\b"
      }
    }
  ]
}
```

</details>

### SACD

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "SACD",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "SACD",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "\\bSACD\\b"
      }
    }
  ]
}
```

</details>

### Cassette

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "Cassette",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Cassette",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "\\bCassette\\b"
      }
    }
  ]
}
```

</details>

### DAT

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "DAT",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "DAT",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "\\bDAT\\b"
      }
    }
  ]
}
```

</details>

### Vinyl

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "Vinyl",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Vinyl",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "\\bVinyl\\b"
      }
    }
  ]
}
```

</details>

### Soundboard

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "Soundboard",
  "includeCustomFormatWhenRenaming": false,
  "specifications": [
    {
      "name": "Soundboard",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": false,
      "fields": {
        "value": "\\bSoundboard\\b"
      }
    }
  ]
}
```

</details>

### 100%

This one is for the Perfect FLAC with a 100% score (so only for CD edition). It can only count in the score while in the search stage, as the torrent's folders and files are not tagged with it.

<details>
<summary>Custom Format JSON</summary>

```json
{
  "name": "100%",
  "includeCustomFormatWhenRenaming": true,
  "specifications": [
    {
      "name": "100%",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": "\\(100%\\)"
      }
    },
    {
      "name": "CD: CD",
      "implementation": "ReleaseTitleSpecification",
      "negate": false,
      "required": true,
      "fields": {
        "value": "\\bCD\\b"
      }
    }
  ]
}
```

</details>

## Quality profile

First, do not forget to edit the **Quality groups**:

![Screenshot 6](https://i.ibb.co/tPmGZvk4/Screenshot-2026-07-11-192219.png)

And choose the **Upgrade Until** flag:

![Screenshot 7](https://i.ibb.co/pmC58NS/Screenshot-2026-07-11-192237.png)

I've decided to give more weight to the encoding quality than to the edition format. 

So I give points to the format like:

*CD|WEB|SACD|DAT* > *Soundboard|Vinyl* > *Cassette*

Let's give them between 5 and 45 points, as we'll use steps of 50 points for the quality:

*FLAC* > *Lossless* > *MP3 TOP* > *Lossy*

Optionally, I'll give the *100%* custom format a score of 5, it will permit to prefer a 100% FLAC CD over any other CD version. Keep in mind that this format will not count once the torrent is grabbed, as explained earlier.

![Screenshot 8](https://i.ibb.co/9ktF28rp/Screenshot-2026-07-11-192319.png)

And finally set the **Upgrade Until Custom Format Score** to 175.

When you'll search a release, the maximum score you'll have with these setings should be :
-  **With 100% at 5 points**: 180 when it's a FLAC 16bit ripped from a CD with 100% score
-  **With 100% at 0 points**: 175 when it's a FLAC 16bit ripped from any digital source

Once grabbed, the maximum score can only be 175 points. Setting a cutoff score to 175 will prevent Lidarr from downloading multiple 100% sources.

## Delay profile

In my scenario, another cause of duplicate downloads is when a 24bit version is announced before the 16bit version. Just add a **Delay profile**, with the timing you prefer, and enable **Bypass if Highest Quality**.

## The result

Reproducing the previous scenario, I choose the same torrent as before:

![Screenshot 9](https://i.ibb.co/673wPhqy/Screenshot-2026-07-11-202346.png)
![Screenshot 10](https://i.ibb.co/LDpdVdGx/Screenshot-2026-07-11-202443.png)

Once imported, the renaming, including the **Format** and **Quality** tokens, finally allows Lidarr to manage as it should other similar releases, by rejecting them.

![Screenshot 11](https://i.ibb.co/Y7TdGzJS/Screenshot-2026-07-11-210226.png)
![Screenshot 12](https://i.ibb.co/Ld0xW753/Screenshot-2026-07-11-210252.png)

You can see here the rejection message.

From that point on, only a Lossless release will be downloaded, and my **Delay Profile** prevents Lidarr from downloading *FLAC 24bit* **AND** *FLAC 16bit* if they are announced in this order within the period of time used.

## FAQ

***Can I manage multiple editions/qualities of the same release in Lidarr?***

Nope, this has to be done manually or by using multiple instances of Lidarr.

***Can I force an edition to be chosen***

Nope, if you force an edition by disabling the [b]Automatically Switch Release** in the release **Edit** modal, it will only prevent Lidarr from importing any edition other than the one you selected, but it will still grab other editions as Lidarr can't analyze a torrent before downloading it.

***I chose the WEB version of an album but Lidarr import it as a Vinyl***

First, check if the wanted edition is on MusicBrainz. If not, please help by adding it!
Even when present, Lidarr can fail to identify the right edition. To help with this, I've just discovered [beets-lidarr](https://github.com/MxMarx/lidarr-beets), I still have to test it and maybe add a longer note about it in a future version of this guide.
