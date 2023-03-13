# Dockerized Robust Tools
A collection of containers for deploying SS14 servers.

## SS14.Watchdog
[Repository](https://github.com/space-wizards/SS14.Watchdog) | [Documentation](https://docs.spacestation14.io/en/getting-started/hosting) | [Docker Hub](https://hub.docker.com/r/morb0/ss14-watchdog)

You can use a Watchdog container to manage multiple instances, but the example below assumes that you use one Watchdog container per server.

**Run:**
```console
$ docker run \
	--mount type=bind,source=/my/path/appsettings.yml,target=/publish/appsettings.yml \
	--mount type=bind,source=/my/path/test-config.toml,target=/publish/instances/test/config.toml \
	-p 5000:80/tcp \
	-p 1121:1121/tcp \
	-p 1121:1121/tcp \
	-p 44880:44880/udp \
	morb0/ss14-watchdog:latest
```

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
      ApiPort: 1121
      # ...
```
*Note: application port can be changed by ASPNETCORE_URLS=http://+:5000*

*Ports can be other if you change example configs, but exposed must be TCP/80 for Watchdog, TCP/1121 for server status API, UDP/1111 for game server and TCP/44880 for metrics*

*But keep port 80 or ASPNETCORE_URLS (if you used this) so server can ping watchdog by localhost in container*

**Example `config.toml`:**
```toml
# ...

[net]
tickrate = 30
port = 1111
bindto = "::,0.0.0.0"
log_late_msg = false

[status]
enabled = true
bind = "*:1121"
connectaddress = "udp://myserverip:1111"

[metrics]
enabled = true
host = "0.0.0.0"
port = 44880

# ...
```


## SS14.Admin
[Repository](https://github.com/space-wizards/SS14.Admin) | Documentation | [Docker Hub](https://hub.docker.com/r/morb0/ss14-admin)

❗ Only can be build on `0982ef3cea8a1e623756433480e0ba4ab8fbc4a3` commit of SS14 repository.  Because current repository submodule version not updated for `PlayTime` feature and latest use unsupported .NET 7 ¯\\\_(ツ)\_/¯

```console
$ cd SS14; git checkout 0982ef3cea8a1e623756433480e0ba4ab8fbc4a3;
```

**Run:**
```console
$ docker run \
	--mount type=bind,source=/my/path/appsettings.yml,target=/publish/appsettings.yml \
	--mount type=volume,source=ss14-admin_data,target=/root/.aspnet/DataProtection-Keys \
	-p 27689:5000 \
	morb0/ss14-admin:latest
```


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


## Robust.Cdn
[Repository](https://github.com/space-wizards/Robust.Cdn) | [Documentation](https://docs.spacestation14.io/en/hosting/robust-cdn) | [Docker Hub](https://hub.docker.com/r/morb0/robust.cdn)

**Run:**
```console
$ docker run \
	--mount type=bind,source=/my/path/appsettings.json,target=/publish/appsettings.json \
	--mount type=bind,source=/my/path/builds,target=/builds \
	--mount type=volume,source=cdn_data,target=/publish/data \
	-p 27690:27690 \
	morb0/robust.cdn:latest
```

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
