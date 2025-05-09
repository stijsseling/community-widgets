## Preview

![](preview.png)
![](preview1.png)


## Configuration

```yaml
- type: custom-api
  title: NBA Today
  url: "https://site.api.espn.com/apis/site/v2/sports/basketball/nba/scoreboard"
  cache: 35s
  template: |
    {{ $events := .JSON.Array "events" }}
    {{ if eq (len $events) 0 }}
      <div>No games scheduled today.</div>
    {{ else }}
      {{ if gt (len $events) 6 }}
        <details style="margin:0;padding:0;border:none">
          <summary style="cursor:pointer;padding:4px 0;font-weight:600">
            Show {{ len $events }} games
          </summary>
      {{ end }}

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

          <span style="display:flex;align-items:flex-start;gap:6px;">
            <span style="display:flex;flex-direction:column;align-items:center;width:24px;">
              <img src="{{ $away.String "team.logo" }}"
                  alt="{{ $away.String "team.abbreviation" }}"
                  style="width:24px;height:24px;"/>
              {{ if and (eq $state "STATUS_IN_PROGRESS") ($g.Exists "competitions.0.situation.possession") }}
                {{ if eq ($g.String "competitions.0.situation.possession") ($away.String "team.id") }}
                  <span style="margin-top:2px;font-size:0.7em;">🏀</span>
                {{ end }}
              {{ end }}
            </span>
            <span style="display:flex;flex-direction:column;">
              <span style="display:flex;align-items:center;">
                {{ $away.String "team.abbreviation" }}
                {{ if ne $state "STATUS_SCHEDULED" }} {{ $away.String "score" }}{{ end }}
              </span>
              <span style="font-size:0.7em;color:var(--glance-muted-text);">
                ({{ $awayRec }})
              </span>
            </span>
          </span>

          <span style="display:flex;flex-direction:column;align-items:center;min-width:70px;">
            <span>
              {{ if eq $state "STATUS_IN_PROGRESS" }}
                {{ $period := $g.String "competitions.0.status.period" }}
                {{ if eq $period "1" }}1st{{ else if eq $period "2" }}2nd{{ else if eq $period "3" }}3rd{{ else if eq $period "4" }}4th{{ else }}OT{{ end }}
                {{ $g.String "competitions.0.status.displayClock" }}
              {{ else if eq $state "STATUS_SCHEDULED" }}
                <span style="font-size:0.75em;">{{ $g.String "competitions.0.status.type.shortDetail" }}</span>
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

          <span style="display:flex;align-items:flex-start;gap:6px;">
            <span style="display:flex;flex-direction:column;align-items:center;width:24px;">
              <img src="{{ $home.String "team.logo" }}"
                  alt="{{ $home.String "team.abbreviation" }}"
                  style="width:24px;height:24px;"/>
              {{ if and (eq $state "STATUS_IN_PROGRESS") ($g.Exists "competitions.0.situation.possession") }}
                {{ if eq ($g.String "competitions.0.situation.possession") ($home.String "team.id") }}
                  <span style="margin-top:2px;font-size:0.7em;">🏀</span>
                {{ end }}
              {{ end }}
            </span>
            <span style="display:flex;flex-direction:column;">
              <span style="display:flex;align-items:center;">
                {{ $home.String "team.abbreviation" }}
                {{ if ne $state "STATUS_SCHEDULED" }} {{ $home.String "score" }}{{ end }}
              </span>
              <span style="font-size:0.7em;color:var(--glance-muted-text);">
                ({{ $homeRec }})
              </span>
            </span>
          </span>

        </li>
      {{ end }}
      </ul>

      {{ if gt (len $events) 6 }}
        </details>
      {{ end }}
    {{ end }}
```
