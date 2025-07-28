# Unraid

Get Docker containers, VM's and/or system information from a local unraid server

Connects to the unraid graphql API. See this page on how to enable the AIP and generate an API key: https://docs.unraid.net/API/how-to-use-the-api/

![](preview.png)

## Docker containers
```yaml
- type: custom-api
  title: UnRaid Docker
  cache: 15m
  url: >-
    http://${UNRAID_HOST}/graphql
  method: POST
  body:
    query: |
      query {
        docker {
          containers {
            names
            ports {
              publicPort
              privatePort
            }
            image
            state
            status
            autoStart
          }
        }
      }
  headers:
    x-api-key: ${UNRAID_API}
    Content-Type: application/json
  template: |
      <ul class="list collapsible-container">
        {{ range sortByString "state" "desc" (.JSON.Array "data.docker.containers") }}
        <li class="flex items-center color-highlight">
          <div class="grow min-width-0">
            <span>{{ .String "names.0" }}</span>
            {{ if eq (.String "state") "RUNNING" }}
              <span class="color-positive">Running</span>
            {{ else if eq (.String "state") "EXITED" }}
              <span class="color-negative">Exited</span>
            {{ else }}
              <span class="color-primary">Unknown</span>
            {{ end }}
            <span class="size-h6">({{ .String "status" }})</span>
          </div>
          {{ if gt (len (.Array "ports")) 0 }}
            {{ range unique "publicPort" (.Array "ports") }}
              <a class="shrink-0 text-right" href=http://${UNRAID_HOST}:{{ .Int "publicPort" }}>{{ .Int "publicPort" }}:{{ .Int "privatePort" }}</a>
            {{ end}}
          {{ end }}
        </li>
        {{ end }}
      </ul>
```

## Virtual Machines
```yaml
- type: custom-api
  title: UnRaid VM's
  cache: 15m
  url: >-
    http://${UNRAID_HOST}/graphql
  method: POST
  body:
    query: |
      query {
        vms {
          domain {
            uuid
            name
            state
          }
        }
      }
  headers:
    x-api-key: ${UNRAID_API}
    Content-Type: application/json
  template: |
      <div class="flex flex-column gap-10">
        <ul class="list collapsible-container" data-collapse-after="5">
          {{ range sortByString "state" "asc" (.JSON.Array "data.vms.domain") }}
          <li class="flex items-center color-highlight">
            <span class="grow min-width-0">{{ .String "name" }}</span>
            <span class="shrink-0 text-right">{{ .String "time" }}</span>
            {{ if eq (.String "state") "RUNNING" }}
                <span class="shrink-0 text-right">Running</span>
              {{ else if eq (.String "state") "SHUTOFF" }}
                <span class="shrink-0 text-right color-negative">Shutoff</span>
              {{ else }}
                <span class="shrink-0 text-right">Unknown</span>
              {{ end }}
          </li>
          {{ end }}
        </ul>
      </div>
```

## Environment variables
- `UNRAID_HOST` - The internal IP-address of the Unraid machine.
- `UNRAID_API` - The API-key of unraid. 
