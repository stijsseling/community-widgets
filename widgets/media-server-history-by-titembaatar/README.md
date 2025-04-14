* [Presentation](#presentation)
* [Preview](#preview)
    * [Full Size Column](#full-size-column)
    * [Small Size Column](#small-size-column)
    * [Compact Mode](#compact-mode)
* [Environment Variables](#environment-variables)
* [User Variables](#user-variables)
* [Widget YAML](#widget-yaml)
    * [Plex YAML](#plex-yaml)
    * [Tautulli YAML](#tautulli-yaml)
    * [Jellyfin YAML](#jellyfin-yaml)

## Presentation

This is a collection of widget to display history from your Media Servers.

Right now I tested `Plex`, `Tautulli` and `Jellyfin`. For `Emby` the API is (from what I glanced)
exactly the same as `Jellyfin`, I think you can use the [Jellyfin Widget](#jellyfin-yaml) with
`Emby` URL and API key. If there is issue, please open an issue and I'll take a closer look.

> [!IMPORTANT]
>
> For `Jellyfin` I couldn't find a way to get the history of all users, so for now it is limited to **ONE** user.

Appearance is the same for all Media Servers. There is some [User Variables](#user-variables)
you can use to display/hide some elements. Please take a look !

## Preview

### Full Size Column

![Preview](preview.png)

### Small Size Column

![Preview Small](preview-small.png)

### Compact Mode

![Preview Compact](preview-compact.png)

## Environment Variables

### Plex

* `PLEX_URL` - the Plex URL, can be `http://<ip_address>:<port>` or `https://<domain>`
* `PLEX_TOKEN` - the Plex token, follow [this guide](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/) if you don't know how to get it

### Tautulli

* `TAUTULLI_URL` - the Tautulli URL, can be `http://<ip_address>:<port>` or `https://<domain>`
* `TAUTULLI_KEY` - the Tautulli API key, can be found in `Settings` -> `Web Interface `-> `API key`

### Jellyfin

* `JELLYFIN_URL` - the Jellyfin URL, can be `http://<ip_address>:<port>` or `https://<domain>`
* `JELLYFIN_KEY` - the Jellyfin API key, use or create one in `Administration` -> `Dashboard` -> `API Keys`
* `JELLYFIN_USER_ID` - the Jellyfin user ID, can be found at `http://localhost:8096/Users?api_key=<your_jellyfin_api_key>`, search for your user and copy the `Id` value

> [!IMPORTANT]
>
> For URLS, you **NEED** to add `http://` or `https://`
> Do **NOT** leave a trailing `/` at the end of your URLs

## User Variables

You can modify some variables inside the `template`. They are inside the block `{{/* USER VARIABLES ... */}}`

* `isSmallColumn` - set to true if using the widget in a small column
* `isCompact` - set to true to use compact mode
* `showThumbnail `- set to true to show thumbnails
* `thumbAspectRatio `- change the thumbnails aspect ratio. Values are `square`, `portrait`, `landscape` or ` `.
* `showUser `- set to true to show user name
* `timeAbsolute `- set to true to user absolute time instead of relative time.

> [!CAUTION]
>
> Displaying the thumbnail **WILL** expose your Token/Api Keys in the HTML.
> Do **NOT** set to true if you are using glance in production or exposing the service to internet.

## Widget YAML

### Plex YAML

> [!NOTE]
>
> In the `parameters` you can change `limit`
> I could not find a way to filter by media types. If you want this feature, consider installing/using Tautulli.

* `limit` - number of most recent played items to return

```yaml
- type: custom-api
  frameless: true
  title: plex history
  cache: 5m
  url: ${PLEX_URL}/status/sessions/history/all
  headers:
    Accept: application/json
    X-Plex-Token: ${PLEX_TOKEN}
  parameters:
    limit: 10 # Modify this value for the length of the history
    sort: viewedAt:desc
  subrequests:
    user:
      url: ${PLEX_URL}/accounts
      headers:
        Accept: application/json
        X-Plex-Token: ${PLEX_TOKEN}
  template: |
    {{/* USER VARIABLES BEGIN */}}

    {{/* Set to true if using the widget in a small column */}}
    {{ $isSmallColumn := false }}

    {{/* Set to true to use a short hand display of Series information */}}
    {{ $isCompact := false }}

    {{/* Set to true to show thumbnails */}}
    {{ $showThumbnail := false }}

    {{/* Depends on $showThumbnail */}}
    {{/* Set to "square" to have an aspect ratio of 1 */}}
    {{/* Set to "portrait" to have an aspect ratio of 3/4 */}}
    {{/* Set to "landscape" to have an aspect ratio of 4/3 */}}
    {{/* Set to "" to have the original aspect ratio */}}
    {{ $thumbAspectRatio := "original" }}

    {{/* Set to true to display user name */}}
    {{ $showUser := true }}

    {{/* Set to true to get absolute time format instead of relatie format */}}
    {{ $timeAbsolute := false }}

    {{/* USER VARIABLES END */}}

    {{ $users := "" }}
    {{ if eq (.Subrequest "user").Response.StatusCode 200 }}
      {{ $users = (.Subrequest "user").JSON.Array "MediaContainer.Account" }}
    {{ end }}

    {{ if eq .Response.StatusCode 200 }}
      {{ $history := .JSON.Array "MediaContainer.Metadata" }}

      {{ if eq (len $history) 0 }}
        <p>stop what you are doing and go watch something !</p>
      {{ else }}
        <div class="carousel-container show-right-cutoff">
          <div class="cards-horizontal carousel-items-container">
            {{ range $n, $item := $history }}
              {{/* PLEX VARIABLES BEGIN */}}

              {{ $user := "" }}
              {{ $userID := $item.Int "accountID" }}
              {{ range $n, $u := $users }}
                {{ if eq $userID ($u.Int "id")}}
                  {{ $user = $u.String "name" }}
                  {{ break }}
                {{ end }}
              {{ end }}

              {{/* PLEX VARIABLES END */}}

              {{/* WIDGET VARIABLES BEGIN */}}

              {{ $mediaType := $item.String "type" }}

              {{ $isMovie := eq $mediaType "movie" }}
              {{ $isShows := eq $mediaType "episode" }}
              {{ $isMusic := eq $mediaType "track" }}

              {{ $movieTitle := $item.String "title" }}
              {{ $showTitle := $item.String "grandparentTitle" }}
              {{ $showSeason := $item.String "parentIndex" }}
              {{ $showEpisode := $item.String "index" }}
              {{ $episodeTitle := $item.String "title" }}
              {{ $artist := $item.String "grandparentTitle" }}
              {{ $albumTitle := $item.String "parentTitle" }}
              {{ $songTitle := $item.String "title" }}
              {{ $default := $item.String "title" }}

              {{ $thumbPath := $item.String "thumb" }}
              {{ if or $isShows $isMusic }}
                {{ $thumbPath = $item.String "parentThumb" }}
              {{ end }}
              {{ $thumbURL := concat "${PLEX_URL}" $thumbPath "?X-Plex-Token=${PLEX_TOKEN}" }}

              {{ $playedAt := $item.String "viewedAt" | parseRelativeTime "unix" }}
              {{ if $timeAbsolute }}
                {{ $t := $item.String "viewedAt" | parseTime "unix" }}
                {{ $playedAt = $t.Format "Jan 02 15:04" }}
              {{ end }}

              {{/* WIDGET VARIABLES END */}}

              {{/* WIDGET TEMPLATE BEGIN */}}

              <div class="card widget-content-frame" >
                {{ if $showThumbnail }}
                  <img
                    src="{{ $thumbURL }}"
                    alt="{{ $default }} thumbnail"
                    loading="lazy"
                    class="shrink-0"
                    style="object-fit: cover;
                      {{ if eq $thumbAspectRatio "square" }}
                        aspect-ratio: 1;
                      {{ else if eq $thumbAspectRatio "portrait" }}
                        aspect-ratio: 3/4;
                      {{ else if eq $thumbAspectRatio "landscape" }}
                        aspect-ratio: 4/3;
                      {{ else }}
                        aspect-ratio: initial;
                      {{ end }}
                      border-radius: var(--border-radius) var(--border-radius) 0 0;"
                  />
                {{ end }}
                <div class="grow padding-inline-widget margin-top-10 margin-bottom-10 {{ if $isSmallColumn -}}text-center{{- end }}" >
                  <ul
                    class="
                      flex
                      flex-column
                      justify-evenly
                      margin-bottom-3
                      {{ if $isSmallColumn -}}size-h6{{- end }}
                    "
                    style="height: 100%;"
                  >
                    {{ if $isCompact }}
                      <ul class="list-horizontal-text flex-nowrap">
                        <li class="color-primary text-truncate">{{ $user }}</li>
                        {{ if not $timeAbsolute }}
                          <li class="shrink-0"><span {{ $playedAt }}></span></li>
                        {{ end }}
                      </ul>
                      {{ if $timeAbsolute }}
                        <li>{{ $playedAt }}</li>
                      {{ end }}
                    {{ else }}
                      {{ if $showUser }}
                        <li class="color-primary text-truncate">{{ $user }}</li>
                      {{ end }}

                      <li class="color-base text-truncate">
                        {{ if $timeAbsolute }}
                          <span>{{ $playedAt }}</span>
                        {{ else }}
                          <span {{ $playedAt }}></span>
                          <span> ago</span>
                        {{ end }}
                      </li>
                    {{ end }}
                    {{ if $isMovie }}
                      <li {{ if $isCompact -}}class="text-truncate"{{- end }}>{{ $movieTitle }}</li>
                    {{ else if $isShows }}
                      {{ if $isCompact }}
                        <ul class="list-horizontal-text flex-nowrap">
                          <li>{{ concat "S" $showSeason "E" $showEpisode }}</li>
                          <li class="text-truncate">{{ $showTitle }}</li>
                        </ul>
                      {{ else }}
                        <li class="text-truncate" >{{ $showTitle }}</li>
                        {{ if $isSmallColumn }}
                          <li>{{ concat "S" $showSeason "E" $showEpisode }}</li>
                        {{ else }}
                          <li class="text-truncate" >{{ concat "Season " $showSeason " Episode " $showEpisode }}</li>
                        {{ end }}
                      {{ end }}
                      <li class="text-truncate" >{{ $episodeTitle }}</li>
                    {{ else if $isMusic }}
                      {{ if $isCompact }}
                        <li class="text-truncate">{{ $artist }}</li>
                      {{ else }}
                        <li class="text-truncate">{{ $artist }}</li>
                        <li class="text-truncate">{{ $albumTitle }}</li>
                      {{ end }}
                      <li class="text-truncate">{{ $songTitle }}</li>
                    {{ else }}
                      <li class="text-truncate">{{ $default }}</li>
                    {{ end }}
                  </ul>
                </div>
              </div>

              {{/* WIDGET TEMPLATE END */}}

            {{ end }}
          </div>
        </div>
      {{ end }}
    {{ else }}
      <p>Failed to fetch Plex history</p>
    {{ end }}
```

### Tautulli YAML

> [!NOTE]
>
> In the `parameters` you can change `media_type` and `length`

* `media_type` - display the media type(s) you want to display, comma-separated. All options are in the inline comment
* `length` - number of most recent played items to return

```yaml
- type: custom-api
  title: tautulli history
  allow-insecure: true
  frameless: true
  cache: 5m
  url: ${TAUTULLI_URL}/api/v2
  parameters:
    apikey: ${TAUTULLI_KEY}
    cmd: get_history
    media_type: movie,episode # movie,episode,track,live,collection,playlist
    length: 10 # Modify this value for the length of the history
  template: |
    {{/* USER VARIABLES BEGIN */}}

    {{/* Set to true if using the widget in a small column */}}
    {{ $isSmallColumn := false }}

    {{/* Set to true to use a short hand display of Series information */}}
    {{ $isCompact := false }}

    {{/* Set to true to show thumbnails */}}
    {{ $showThumbnail := false }}

    {{/* Depends on $showThumbnail */}}
    {{/* Set to "square" to have an aspect ratio of 1 */}}
    {{/* Set to "portrait" to have an aspect ratio of 3/4 */}}
    {{/* Set to "landscape" to have an aspect ratio of 4/3 */}}
    {{/* Set to "" to have the original aspect ratio */}}
    {{ $thumbAspectRatio := "original" }}

    {{/* Set to true to display user name */}}
    {{ $showUser := true }}

    {{/* Set to true to get absolute time format instead of relatie format */}}
    {{ $timeAbsolute := false }}

    {{/* USER VARIABLES END */}}

    {{ if eq .Response.StatusCode 200 }}
      {{ $history := .JSON.Array "response.data.data" }}

      {{ if eq (len $history) 0 }}
        <div class="card widget-content-frame padding-widget">
          <p>stop what you are doing and go watch something !</p>
        </div>
      {{ else }}
        <div class="carousel-container show-right-cutoff">
          <div class="cards-horizontal carousel-items-container">
            {{ range $n, $item := $history }}

              {{/* WIDGET VARIABLES BEGIN */}}

              {{ $user := $item.String "user" }}

              {{ $mediaType := $item.String "media_type" }}

              {{ $isMovie := eq $mediaType "movie" }}
              {{ $isShows := eq $mediaType "episode" }}
              {{ $isMusic := eq $mediaType "track" }}

              {{ $movieTitle := $item.String "title" }}
              {{ $showTitle := $item.String "grandparent_title" }}
              {{ $showSeason := $item.String "parent_media_index" }}
              {{ $showEpisode := $item.String "media_index" }}
              {{ $episodeTitle := $item.String "title" }}
              {{ $artist := $item.String "grandparent_title" }}
              {{ $albumTitle := $item.String "parent_title" }}
              {{ $songTitle := $item.String "title" }}
              {{ $default := $item.String "title" }}

              {{ $thumbPath := $item.String "thumb" }}
              {{ $thumbURL := concat "${TAUTULLI_URL}/api/v2?apikey=${TAUTULLI_KEY}&cmd=pms_image_proxy&img=" $thumbPath }}

              {{ $playedAt := $item.String "date" | parseRelativeTime "unix" }}
              {{ if $timeAbsolute }}
                {{ $t := $item.String "date" | parseTime "unix" }}
                {{ $playedAt = $t.Format "Jan02 15:04" }}
              {{ end }}

              {{/* WIDGET VARIABLES END */}}

              {{/* WIDGET TEMPLATE BEGIN */}}

              <div class="card widget-content-frame" >
                {{ if $showThumbnail }}
                  <img
                    src="{{ $thumbURL }}"
                    alt="{{ $default }} thumbnail"
                    loading="lazy"
                    class="shrink-0"
                    style="object-fit: cover;
                      {{ if eq $thumbAspectRatio "square" }}
                        aspect-ratio: 1;
                      {{ else if eq $thumbAspectRatio "portrait" }}
                        aspect-ratio: 3/4;
                      {{ else if eq $thumbAspectRatio "landscape" }}
                        aspect-ratio: 4/3;
                      {{ else }}
                        aspect-ratio: initial;
                      {{ end }}
                      border-radius: var(--border-radius) var(--border-radius) 0 0;"
                  />
                {{ end }}
                <div class="grow padding-inline-widget margin-top-10 margin-bottom-10 {{ if $isSmallColumn -}}text-center{{- end }}" >
                  <ul
                    class="
                      flex
                      flex-column
                      justify-evenly
                      margin-bottom-3
                      {{ if $isSmallColumn -}}size-h6{{- end }}
                    "
                    style="height: 100%;"
                  >
                    {{ if $isCompact }}
                      <ul class="list-horizontal-text flex-nowrap">
                        <li class="color-primary text-truncate">{{ $user }}</li>
                        {{ if not $timeAbsolute }}
                          <li class="shrink-0"><span {{ $playedAt }}></span></li>
                        {{ end }}
                      </ul>
                      {{ if $timeAbsolute }}
                        <li>{{ $playedAt }}</li>
                      {{ end }}
                    {{ else }}
                      {{ if $showUser }}
                        <li class="color-primary text-truncate">{{ $user }}</li>
                      {{ end }}

                      <li class="color-base text-truncate">
                        {{ if $timeAbsolute }}
                          <span>{{ $playedAt }}</span>
                        {{ else }}
                          <span {{ $playedAt }}></span>
                          <span> ago</span>
                        {{ end }}
                      </li>
                    {{ end }}
                    {{ if $isMovie }}
                      <li {{ if $isCompact -}}class="text-truncate"{{- end }}>{{ $movieTitle }}</li>
                    {{ else if $isShows }}
                      {{ if $isCompact }}
                        <ul class="list-horizontal-text flex-nowrap">
                          <li>{{ concat "S" $showSeason "E" $showEpisode }}</li>
                          <li class="text-truncate">{{ $showTitle }}</li>
                        </ul>
                      {{ else }}
                        <li class="text-truncate" >{{ $showTitle }}</li>
                        {{ if $isSmallColumn }}
                          <li>{{ concat "S" $showSeason "E" $showEpisode }}</li>
                        {{ else }}
                          <li class="text-truncate" >{{ concat "Season " $showSeason " Episode " $showEpisode }}</li>
                        {{ end }}
                      {{ end }}
                      <li class="text-truncate" >{{ $episodeTitle }}</li>
                    {{ else if $isMusic }}
                      {{ if $isCompact }}
                        <li class="text-truncate">{{ $artist }}</li>
                      {{ else }}
                        <li class="text-truncate">{{ $artist }}</li>
                        <li class="text-truncate">{{ $albumTitle }}</li>
                      {{ end }}
                      <li class="text-truncate">{{ $songTitle }}</li>
                    {{ else }}
                      <li class="text-truncate">{{ $default }}</li>
                    {{ end }}
                  </ul>
                </div>
              </div>

              {{/* WIDGET TEMPLATE END */}}

            {{ end }}
          </div>
        </div>
      {{ end }}
    {{ else }}
      <p>Failed to fetch Plex history</p>
    {{ end }}
```

### Jellyfin YAML

> [!NOTE]
>
> In the `parameters` you can change `Limit` and `IncludeItemTypes`

* `Limit` - number of most recent played items to return
* `IncludeItemTypes` - display the media type(s) you want to display, comma-separated. All options bellow (there is a lot and you probably can stick wihth `Movie`, `Episode` and `Audio` )

<details>
<summary>Available media types</summary>
`AggregateFolder` `Audio` `AudioBook` `BasePluginFolder` `Book` `BoxSet` `Channel` `ChannelFolderItem` `CollectionFolder` `Episode` `Folder` `Genre` `ManualPlaylistsFolder` `Movie` `LiveTvChannel` `LiveTvProgram` `MusicAlbum` `MusicArtist` `MusicGenre` `MusicVideo` `Person` `Photo` `PhotoAlbum` `Playlist` `PlaylistsFolder` `Program` `Recording` `Season` `Series` `Studio` `Trailer` `TvChannel` `TvProgram` `UserRootFolder` `UserView` `Video` `Year`
</details>

```yaml
- type: custom-api
  frameless: true
  title: jellyfin history
  cache: 5m
  url: ${JELLYFIN_URL}/Users/${JELLYFIN_USER_ID}/Items
  parameters:
    api_key: ${JELLYFIN_KEY}
    Limit: 10 # Modify this value for the length of the history
    IncludeItemTypes: Movie,Episode # Movie,Episode,Audio,Playlist... Too much, read the README.md
    Recursive: true
    isPlayed: true
    SortBy: DatePlayed
    SortOrder: Descending
  subrequests:
    user:
      url: ${JELLYFIN_URL}/Users/${JELLYFIN_USER_ID}
      parameters:
        api_key: ${JELLYFIN_KEY}
  template: |
    {{/* USER VARIABLES BEGIN */}}

    {{/* Set to true if using the widget in a small column */}}
    {{ $isSmallColumn := false }}

    {{/* Set to true to use a short hand display of Series information */}}
    {{ $isCompact := false }}

    {{/* Set to true to show thumbnails */}}
    {{ $showThumbnail := false }}

    {{/* Depends on $showThumbnail */}}
    {{/* Set to "square" to have an aspect ratio of 1 */}}
    {{/* Set to "portrait" to have an aspect ratio of 3/4 */}}
    {{/* Set to "landscape" to have an aspect ratio of 4/3 */}}
    {{/* Set to "" to have the original aspect ratio */}}
    {{ $thumbAspectRatio := "original" }}

    {{/* Set to true to display user name */}}
    {{ $showUser := true }}

    {{/* Set to true to get absolute time format instead of relatie format */}}
    {{ $timeAbsolute := false }}

    {{/* USER VARIABLES END */}}

    {{ $user := (.Subrequest "user").JSON.String "Name" }}

    {{ if eq .Response.StatusCode 200 }}
      {{ $history := .JSON.Array "Items" }}

      {{ if eq (len $history) 0 }}
        <p>stop what you are doing and go watch something !</p>
      {{ else }}
        <div class="carousel-container show-right-cutoff">
          <div class="cards-horizontal carousel-items-container">
            {{ range $n, $item := $history }}
              {{/* WIDGET VARIABLES BEGIN */}}

              {{ $mediaType := $item.String "Type" }}

              {{ $isMovie := eq $mediaType "Movie" }}
              {{ $isShows := eq $mediaType "Episode" }}
              {{ $isMusic := eq $mediaType "Audio" }}

              {{ $movieTitle := $item.String "Name" }}
              {{ $showTitle := $item.String "SeriesName" }}
              {{ $showSeason := $item.String "ParentIndexNumber" }}
              {{ $showEpisode := $item.String "IndexNumber" }}
              {{ $episodeTitle := $item.String "Name" }}
              {{ $artist := $item.String "AlbumArtist" }}
              {{ $albumTitle := $item.String "Album" }}
              {{ $songTitle := $item.String "Name" }}
              {{ $default := $item.String "Name" }}

              {{ $thumbID := $item.String "Id" }}
              {{ if $isShows }}
                {{ $thumbID = $item.String "SeasonId" }}
              {{ end }}
              {{ $thumbURL := concat "${JELLYFIN_URL}/Items/" $thumbID "/Images/Primary?api_key=${JELLYFIN_KEY}" }}

              {{ $playedAt := $item.String "UserData.LastPlayedDate" | parseRelativeTime "rfc3339" }}
              {{ if $timeAbsolute }}
                {{ $t := $item.String "UserData.LastPlayedDate" | parseTime "rfc3339" }}
                {{ $playedAt = $t.Format "Jan 02 15:04" }}
              {{ end }}

              {{/* WIDGET VARIABLES END */}}

              {{/* WIDGET TEMPLATE BEGIN */}}

              <div class="card widget-content-frame" >
                {{ if $showThumbnail }}
                  <img
                    src="{{ $thumbURL }}"
                    alt="{{ $default }} thumbnail"
                    loading="lazy"
                    class="shrink-0"
                    style="object-fit: cover;
                      {{ if eq $thumbAspectRatio "square" }}
                        aspect-ratio: 1;
                      {{ else if eq $thumbAspectRatio "portrait" }}
                        aspect-ratio: 3/4;
                      {{ else if eq $thumbAspectRatio "landscape" }}
                        aspect-ratio: 4/3;
                      {{ else }}
                        aspect-ratio: initial;
                      {{ end }}
                      border-radius: var(--border-radius) var(--border-radius) 0 0;"
                  />
                {{ end }}
                <div class="grow padding-inline-widget margin-top-10 margin-bottom-10 {{ if $isSmallColumn -}}text-center{{- end }}" >
                  <ul
                    class="
                      flex
                      flex-column
                      justify-evenly
                      margin-bottom-3
                      {{ if $isSmallColumn -}}size-h6{{- end }}
                    "
                    style="height: 100%;"
                  >
                    {{ if $isCompact }}
                      <ul class="list-horizontal-text flex-nowrap">
                        <li class="color-primary text-truncate">{{ $user }}</li>
                        {{ if not $timeAbsolute }}
                          <li class="shrink-0"><span {{ $playedAt }}></span></li>
                        {{ end }}
                      </ul>
                      {{ if $timeAbsolute }}
                        <li>{{ $playedAt }}</li>
                      {{ end }}
                    {{ else }}
                      {{ if $showUser }}
                        <li class="color-primary text-truncate">{{ $user }}</li>
                      {{ end }}

                      <li class="color-base text-truncate">
                        {{ if $timeAbsolute }}
                          <span>{{ $playedAt }}</span>
                        {{ else }}
                          <span {{ $playedAt }}></span>
                          <span> ago</span>
                        {{ end }}
                      </li>
                    {{ end }}
                    {{ if $isMovie }}
                      <li {{ if $isCompact -}}class="text-truncate"{{- end }}>{{ $movieTitle }}</li>
                    {{ else if $isShows }}
                      {{ if $isCompact }}
                        <ul class="list-horizontal-text flex-nowrap">
                          <li>{{ concat "S" $showSeason "E" $showEpisode }}</li>
                          <li class="text-truncate">{{ $showTitle }}</li>
                        </ul>
                      {{ else }}
                        <li class="text-truncate" >{{ $showTitle }}</li>
                        {{ if $isSmallColumn }}
                          <li>{{ concat "S" $showSeason "E" $showEpisode }}</li>
                        {{ else }}
                          <li class="text-truncate" >{{ concat "Season " $showSeason " Episode " $showEpisode }}</li>
                        {{ end }}
                      {{ end }}
                      <li class="text-truncate" >{{ $episodeTitle }}</li>
                    {{ else if $isMusic }}
                      {{ if $isCompact }}
                        <li class="text-truncate">{{ $artist }}</li>
                      {{ else }}
                        <li class="text-truncate">{{ $artist }}</li>
                        <li class="text-truncate">{{ $albumTitle }}</li>
                      {{ end }}
                      <li class="text-truncate">{{ $songTitle }}</li>
                    {{ else }}
                      <li class="text-truncate">{{ $default }}</li>
                    {{ end }}
                  </ul>
                </div>
              </div>

              {{/* WIDGET TEMPLATE END */}}

            {{ end }}
          </div>
        </div>
      {{ end }}
    {{ else }}
      <p>Failed to fetch Jellyfin history</p>
    {{ end }}
```
