## Preview

![](preview.png)

## Configuration

```yaml
type: custom-api
title: NBA Today
url: "https://site.api.espn.com/apis/site/v2/sports/basketball/nba/scoreboard"
cache: 20s
template: |
  {{ $events := .JSON.Array "events" }}
  {{ if eq (len $events) 0 }}
    <div>No games scheduled today.</div>
  {{ else }}
    <ul style="list-style:none;padding:0;margin:0;">
    {{ range $events }}
      {{ $g := . }}
      {{ $state := $g.String "competitions.0.status.type.name" }}
      {{ $away := index ($g.Array "competitions.0.competitors") 0 }}
      {{ $home := index ($g.Array "competitions.0.competitors") 1 }}
      {{ $awayRec := (index ($away.Array "records") 0).String "summary" }}
      {{ $homeRec := (index ($home.Array "records") 0).String "summary" }}
      <li
        style="display:flex;align-items:center;white-space:nowrap;gap:12px;padding:4px 0;cursor:default"
        {{ if ne $state "STATUS_SCHEDULED" }}title="
          {{ $away.String "team.abbreviation" }} Box:{{ range $i,$ls := $away.Array "linescores" }}{{ if eq $i 0 }} Q1: {{ else if eq $i 1 }} Q2: {{ else if eq $i 2 }} Q3: {{ else if eq $i 3 }} Q4: {{ else }} OT: {{ end }}{{$ls.String "value"}}{{ end }}&#10;
          {{ $home.String "team.abbreviation" }} Box:{{ range $i,$ls := $home.Array "linescores" }}{{ if eq $i 0 }} Q1: {{ else if eq $i 1 }} Q2: {{ else if eq $i 2 }} Q3: {{ else if eq $i 3 }} Q4: {{ else }} OT: {{ end }}{{$ls.String "value"}}{{ end }}
        "{{ end }}>

        <span style="display:flex;flex-direction:column;align-items:flex-start;">
          <span style="display:flex;align-items:center;">
            <img src="{{ $away.String "team.logo" }}"
                alt="{{ $away.String "team.abbreviation" }}"
                style="width:24px;height:24px;margin-right:6px;"/>
            {{ $away.String "team.abbreviation" }} {{ $away.String "score" }}
          </span>
          <span style="font-size:0.7em;color:var(--glance-muted-text);margin-left:30px;">
            ({{ $awayRec }})
          </span>
        </span>

        <span style="display:flex;flex-direction:column;align-items:center;">
          <span>
            {{ if eq $state "STATUS_IN_PROGRESS" }}
              {{ with $g.Int "competitions.0.status.period" }}
                {{ if eq . 1 }}1st{{ else if eq . 2 }}2nd{{ else if eq . 3 }}3rd{{ else if eq . 4 }}4th{{ else }}OT{{ end }}
              {{ end }} {{ $g.String "competitions.0.status.displayClock" }}
            {{ else }}
              {{ $g.String "competitions.0.status.type.shortDetail" }}
            {{ end }}
          </span>
          {{ if $g.Exists "competitions.0.series" }}
            <span style="font-size:0.7em;color:var(--glance-accent-color);">
              {{ $g.String "competitions.0.series.summary" }}
            </span>
          {{ end }}
        </span>

        <span style="display:flex;flex-direction:column;align-items:flex-start;">
          <span style="display:flex;align-items:center;">
            <img src="{{ $home.String "team.logo" }}"
                alt="{{ $home.String "team.abbreviation" }}"
                style="width:24px;height:24px;margin-right:6px;"/>
            {{ $home.String "team.abbreviation" }} {{ $home.String "score" }}
          </span>
          <span style="font-size:0.7em;color:var(--glance-muted-text);margin-left:30px;">
            ({{ $homeRec }})
          </span>
        </span>

      </li>
    {{ end }}
    </ul>
  {{ end }}
```
