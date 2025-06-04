![](preview.png)

```yaml
- type: custom-api
  title: Trending
  title-url: ${OVERSEERR_URL}/discover/trending    # trending, movies, or tv
  url: ${OVERSEERR_URL}/api/v1/discover/trending   # trending, movies, or tv
  cache: 15m
  headers:
    accept: "application/json"
    x-api-key: ${OVERSEERR_API_KEY}
  parameters:
    language: "en" # this can be changed to other languages if desired
    page: "1"
  template: |
    {{ if eq .Response.StatusCode 200 }}
      {{ $items := .JSON.Array "results" }}
      {{ if gt (len $items) 0 }}
        <ul class="list list-gap-10 collapsible-container" data-collapse-after="5">
        {{ range $items }}
          <li class="flex items-start gap-10 thumbnail-container thumbnail-parent">
            {{ $mediaType := .String "mediaType" }}
            {{ $mediaTitle := "" }}
            {{ $overseerrUrl := concat "${OVERSEERR_URL}" "/" $mediaType "/" ( .String "id" ) }}
            {{ $tmdbUrl := concat "https://www.themoviedb.org" "/" $mediaType "/" ( .String "id" ) }}
            {{ if eq $mediaType "movie" }} {{ $mediaTitle = .String "title" }}{{ else }}{{ $mediaTitle = .String "name" }}{{ end }}
            <a href={{ $overseerrUrl }} target="_blank">
            <img src={{ concat "https://image.tmdb.org/t/p/w300" (.String "posterPath") }} style="border-radius: 5px; min-width: 5rem; max-width: 5rem;" class="card">
            </a>
            <div class="flex-1 text-truncate">
              <p class="color-positive size-h4 margin-top-5" title={{ $mediaTitle }}><a href={{ $overseerrUrl }} target="_blank">{{ $mediaTitle }}</p>
              <p class="color-subdue size-h4" title="TMDB Rating"><a href={{ $tmdbUrl }} target="_blank">TMDB: {{ mul (.Float "voteAverage") 10 | toInt }}%</a></p>
              <p class="color-subdue size-h4" title={{ .String "overview" }}>{{ .String "overview" }}</p>
            </div>
          </li>
        {{ end }}
        </ul>
      {{ else }}
        <details style="margin-top:0.5rem">
          <summary> Raw JSON:</summary>
          <pre stye="font-size:10px">
      {{ (printf "%+v" .JSON) }}
          </pre>
        </details>
      {{ end }}
    {{ end }}
```

## Environment variables

- `OVERSEERR_URL` - the URL of the Overseerr instance, please include `http://` or `https://` (preferred: `https://`)
- `OVERSEERR_API_KEY` - the API key of the server which can be found in `Overseerr settings -> General -> API key`
