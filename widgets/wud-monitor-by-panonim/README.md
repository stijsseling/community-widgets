# What's Up Docker Monitor v1.1
This widget uses WUD api. It fetches all the containers and displayes them in Glance. It checks if container needs an update and displayes it. You can also decided if you want to toggle displaying all the container or only one's that needs an update.

**Note:** In newest update WUD Monitor change api fetching from `POST` to `GET`, because many people complained about long waiting times. If you don't like this change change code as shown below. What's the diffrence? Widget now relays on WUD to update the api instead of forcing it, so now updating the information will be slower.
```txt
url: http://${WUD_URL}/api/containers/watch
method: POST
```

To toggle showing all containers, you need to set the variable `$showAll` to `true`. You can do this by changing the line `{{ $showAll := false }}` in the code to `{{ $showAll := true }}`. Setting it to true will also make sure that images needing an update will be displayed on top. This will display all containers, regardless of whether they need an update or not.

There's also a toggle to turn on/off a message indicating that all containers are Up-To-Date `{{ $hasUpdates := false }}`.
```yaml
       - type: custom-api
         title: What's Up Docker?
         cache: 1h
         url: http://${WUD_URL}/api/containers
         method: GET
         template: |
           <ul class="list list-gap-10 collapsible-container" data-collapse-after="3">
             {{ $showAll := false }}  {{/* Set this to true to show all containers */}}
             {{ $containers := .JSON.Array "" }}
             {{ $hasUpdates := false }} {{/* Set to true to hide up-to-date message */}}
             {{ range $index, $container := $containers }}
               {{ if $container.Bool "updateAvailable" }}
                 {{ $hasUpdates = true }}
                 <li>
                   <a class="size-h4 color-highlight block text-truncate" href="https://hub.docker.com/r/{{ $container.String "image.name" }}" target="_blank">{{ $container.String "name" }}</a>
                   <ul class="list-horizontal">
                     <li>Status:
                       {{ if eq ( $container.String "status" ) "running" }}
                         <span class="color-positive">●</span> Running
                       {{ else }}
                         <span class="color-negative">●</span> Not Running
                       {{ end }}
                     </li>
                     <li>Watcher: {{ $container.String "watcher" }}</li>
                     <li>Update Available: <span class="color-positive">Yes</span></li>
                   </ul>
                 </li>
               {{ end }}
             {{ end }}
             {{ if not $hasUpdates }}
               <li><span class="color-positive">You're good to go!</span></li>
             {{ end }}
             {{ if $showAll }}
               {{ range $index, $container := $containers }}
                 {{ if not ( $container.Bool "updateAvailable" ) }}
                   <li>
                    {{ $registryName := $container.String "image.registry.name" }}
                      {{ $imageName := $container.String "image.name" }}
                      {{ $hubSource := "#" }}
                      {{ if eq $registryName "hub.public" }}
                        {{ $hubSource = concat "https://hub.docker.com/r/" $imageName }}
                      {{ else if eq $registryName "ghcr.public" }}
                        {{ $hubSource = concat "https://github.com/" $imageName }}
                      {{ end }}
                      <a class="size-h4 color-highlight block text-truncate" href="{{ $hubSource }}" target="_blank" rel="noreferrer">{{ $container.String "name" }}</a>
                     <ul class="list-horizontal">
                       <li>Status:
                         {{ if eq ( $container.String "status" ) "running" }}
                           <span class="color-positive">●</span> Running
                         {{ else }}
                           <span class="color-negative">●</span> Not Running
                         {{ end }}
                       </li>
                       <li>Watcher: {{ $container.String "watcher" }}</li>
                       <li>Update Available: <span class="color-negative">No</span></li>
                     </ul>
                   </li>
                 {{ end }}
               {{ end }}
             {{ end }}
           </ul>
```
## Environment variables
`WUD_URL` - the URL of the Whats up docker server

Template: `WUD_URL=ip:port` - You can also just reaplace the code var for it to work. 

For grabbing container no matter the state I also recommend adding this to your WUD env:
```txt
- WUD_WATCHER_LOCAL_WATCHALL=true
```
Please remember to restart your services after applying env vars.

## Preview
[![showAll var = false](./preview1.png)](./preview1.png)

[![showAll var = true](./preview_2.png)](./preview2.png)
<hr>
Made by: Artur Flis

Contact: @blue.dev on Project's Discord.

<hr>

### Contributors v1.1

- [**ᴠᴀʀɪᴀʙʟᴇ**](https://github.com/ralphocdol) – Improving linking container images directly to their source (Docker Hub or GitHub).

