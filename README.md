# Dockerized Robust Tools
*//TODO: Description*

## SS14.Watchdog
[Repository](https://github.com/space-wizards/SS14.Watchdog) | [Documentation](https://docs.spacestation14.io/en/getting-started/hosting) | [Docker Hub](https://hub.docker.com/r/morb0/watchdog)

**Run:**
```console
$ docker run \
	--mount type=bind,source=/opt/watchdog/appsettings.yml,target=/publish/appsettings.yml \
	--mount type=bind,source=/var/lib/wizards/instances,target=/publish/instances \
	--port 80:5000 \
	morb0/watchdog:latest
```
Where */opt/watchdog/appsettings.json* and */var/lib/wizards/instances* is your host directories.

**Example `appsettings.yml`:**
```yml
Serilog:
  Using: [ "Serilog.Sinks.Console", "Serilog.Sinks.Loki" ]
  MinimumLevel:
    Default: Information
    Override:
      SS14: Debug
      Microsoft: "Warning"
      Microsoft.Hosting.Lifetime: "Information"
      Microsoft.AspNetCore: Warning

  WriteTo:
    - Name: Console
      Args:
        OutputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3} {SourceContext}] {Message:lj}{NewLine}{Exception}"

  Enrich: [ "FromLogContext" ]

BaseUrl: "http://0.0.0.0:80/"

AllowedHosts: "*"

Servers:
  Instances:
    test:
      Name: "Test Server"
      ApiToken: "mysecrettoken"
      ApiPort: 1212
      # ...
```


## Robust.Cdn
[Repository](https://github.com/space-wizards/Robust.Cdn) | [Documentation](https://docs.spacestation14.io/en/hosting/robust-cdn) | [Docker Hub](https://hub.docker.com/r/morb0/robust.cdn)

**Run:**
```console
$ docker run \
	--mount type=bind,source=/opt/cdn/appsettings.json,target=/publish/appsettings.json \
	--mount type=bind,source=/var/lib/wizards-builds/builds,target=/builds \
	--mount type=volume,source=cdn_data,target=/publish/data \
	--port 27690:27690 \
	morb0/robust.cdn:latest
```
Where */opt/cdn/appsettings.json* and */var/lib/wizards-builds/builds* is your host directories.
And *cdn_data* is Docker volume.

**Example `appsettings.json`:**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Urls": "http://0.0.0.0:27690/",
  "Cdn": {
    "UpdateToken": "mysecrettoken",
    "StreamCompressLevel": 5,
    "DatabaseFileName": "data/content.db",
    "VersionDiskPath": "/builds"
  }
}
```
