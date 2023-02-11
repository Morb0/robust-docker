# Dockerized Robust Tools
*//TODO: Description*

## Robust.Cdn
[Repository](https://github.com/space-wizards/Robust.Cdn) | [Documentation](https://docs.spacestation14.io/en/hosting/robust-cdn) | [Docker Hub](https://hub.docker.com/r/morb0/robust.cdn)

**Run:**
```console
$ docker run \
	--mount type=bind,source=/opt/cdn/appsettings.json,target=/publish/appsettings.json \
	--mount type=bind,source=/var/lib/wizards-builds/builds,target=/builds \
	--mount type=volume,source=cdn_data,target=/publish/content.db \
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
