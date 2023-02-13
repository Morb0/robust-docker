# Dockerized Robust Tools
*//TODO: Description*

## SS14.Admin
[Repository](https://github.com/space-wizards/SS14.Admin) | Documentation | [Docker Hub](https://hub.docker.com/r/morb0/ss14-admin)

❗ Only can be build on `0982ef3cea8a1e623756433480e0ba4ab8fbc4a3` commit of SS14 repository.  Because current repository submodule version not updated for `PlayTime` feature and latest use unsupported .NET 7 ¯\\\_(ツ)\_/¯

```console
$ cd SS14; git checkout 0982ef3cea8a1e623756433480e0ba4ab8fbc4a3;
```

**Run:**
```console
$ docker run \
	--mount type=bind,source=/opt/admin/appsettings.yml,target=/publish/appsettings.yml \
	-v ss14-admin_data:/root/.aspnet/DataProtection-Keys \
	-p 27689:5000 \
	morb0/ss14_admin:latest
```
Where */opt/admin/appsettings.yml* is your host directory.


**Example `appsettings.yml`:**
```yml
Serilog:
  Using: [ "Serilog.Sinks.Console" ]
  MinimumLevel:
    Default: Information
    Override:
      SS14: Debug
      Microsoft: "Warning"
      Microsoft.Hosting.Lifetime: "Information"
      Microsoft.AspNetCore: Warning
      IdentityServer4: Warning

  WriteTo:
    - Name: Console
      Args:
        OutputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3} {SourceContext}] {Message:lj}{NewLine}{Exception}"
  Enrich: [ "FromLogContext" ]

ConnectionStrings:
  # Connects to the same postgres database as the game server
  DefaultConnection: "Server=<db_ip>;Port=<db_port>;Database=<db_name>;User Id=<db_user>;Password=<db_pass>"

AllowedHosts: "*"

urls: "http://localhost:27689/"

PathBase: "/"

WebRootPath: "/publish/wwwroot"

ForwardProxies:
    - 127.0.0.1

Auth:
  Authority: "https://central.spacestation14.io/web/"
  ClientId: "myclientid"
  ClientSecret: "myclientsecret"

authServer: "https://central.spacestation14.io/auth"
```

**Example Nginx config:**
```nginx
location /admin/ {
	proxy_pass          http://localhost:27689;
	proxy_http_version  1.1;
	proxy_set_header    Upgrade $http_upgrade;
	proxy_set_header    Connection keep-alive;
	proxy_set_header    Host $host;
	proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header    X-Forwarded-Proto https;
	proxy_cache_bypass  $http_upgrade;

	# Necessary to avoid errors from too large headers thanks to large cookies.
	proxy_buffer_size        128k;
	proxy_buffers            4 256k;
	proxy_busy_buffers_size  256k;
}
```


## SS14.Watchdog
[Repository](https://github.com/space-wizards/SS14.Watchdog) | [Documentation](https://docs.spacestation14.io/en/getting-started/hosting) | [Docker Hub](https://hub.docker.com/r/morb0/ss14-watchdog)

**Run:**
```console
$ docker run \
	--mount type=bind,source=/opt/watchdog/appsettings.yml,target=/publish/appsettings.yml \
	--mount type=bind,source=/opt/ss14_instances,target=/publish/instances \
	-p 80:5000 \
	morb0/watchdog:latest
```
Where */opt/watchdog/appsettings.json* and */opt/ss14_instances* is your host directories.

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

BaseUrl: "http://localhost:80/"

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
	-p 27690:27690 \
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
