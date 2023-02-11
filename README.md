# Dockerized Robust Tools
*//TODO: Description*

## [Robust.Cdn](https://github.com/space-wizards/Robust.Cdn)
Documentation:
https://docs.spacestation14.io/en/hosting/robust-cdn

Run:
```sh
$ docker run \
	--mount type=bind,source=/opt/cdn/appsettings.json,target=/publish/appsettings.json \
	--mount type=bind,source=/var/lib/wizards-builds/builds,target=/builds \
	--port 27690:27690
	morb0/robust.cdn:latest
```

Example `appsettings.json`:
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Urls": "http://localhost:27690/",
  "Cdn": {
    "UpdateToken": "mysecrettoken",
    "StreamCompressLevel": 5,
    "DatabaseFileName": "content.db",
    "VersionDiskPath": "/builds"
  }
}
```
