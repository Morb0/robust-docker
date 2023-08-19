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
	-p 1111:1111/udp \
	-p 44880:44880/tcp \
	morb0/ss14-watchdog:latest
```

<details>
  <summary>Example appsettings.yml:</summary>

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
</details>
*Note: application port can be changed by ASPNETCORE_URLS=http://+:5000*

*Ports can be other if you change example configs, but exposed must be TCP/80 for Watchdog, TCP/1121 for server status API, UDP/1111 for game server and TCP/44880 for metrics*

*But keep port 80 or ASPNETCORE_URLS (if you used this) so server can ping watchdog by localhost in container*

<details>
  <summary>Example config.toml:</summary>

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
</details>

## SS14.Admin
[Repository](https://github.com/space-wizards/SS14.Admin) | Documentation | [Docker Hub](https://hub.docker.com/r/morb0/ss14-admin)

**Run:**
```console
$ docker run \
	--mount type=bind,source=/my/path/appsettings.yml,target=/publish/appsettings.yml \
	--mount type=volume,source=ss14-admin_data,target=/root/.aspnet/DataProtection-Keys \
	-p 27689:5000 \
	morb0/ss14-admin:latest
```


<details>
  <summary>Example appsettings.json:</summary>

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
</details>

<details>
  <summary>Example Nginx config:</summary>

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
</details>

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

<details>
  <summary>Example appsettings.json:</summary>

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
</details>

## SS14.Changelog
[Repository](https://github.com/space-wizards/SS14.Changelog) | [Documentation](https://docs.spacestation14.io/en/hosting/changelogs) | [Docker Hub](https://hub.docker.com/r/morb0/ss14-changelog)

**Run:**
```console
$ docker run \
	--mount type=bind,source=/my/path/appsettings.json,target=/publish/appsettings.json \
        --mount type=bind,source=/my/path/ssh_key,target=/publish/ssh_key \
	--mount type=volume,source=repo_data,target=/publish/repo \
	-p 4566:4566 \
	morb0/ss14-changelog:latest
```

<details>
  <summary>Example appsettings.json:</summary>

```yaml
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

AllowedHosts: "*"

urls: "http://localhost:4566"

Changelog:
  GitHubSecret: "secretGH"
  ChangelogBranchName: "master"
  ChangelogFilename: "ChangelogMyFork.yml"
  SshKey: "/publish/ssh_key"
  ChangelogRepo: "/publish/repo"
  DelaySeconds: 60
```
</details>

## SS14.MapServer
[Repository](https://github.com/juliangiebel/SS14.MapServer) | Documentation | [Docker Hub](https://hub.docker.com/r/morb0/ss14-mapserver)

**Run:**
```console
$ docker run \
	--mount type=bind,source=/my/path/appsettings.yml,target=/app/appsettings.yaml \
        --mount type=bind,source=/my/path/private-key.pem,target=/app/private-key.pem \
	--mount type=volume,source=ss14-mapserver_data,target=/app/build \
	-p 5000:80 \
	morb0/ss14-mapserver:latest
```

<details>
  <summary>Example appsettings.json:</summary>

```yaml
Serilog:
  Using: [ "Serilog.Sinks.Console" ]
  MinimumLevel:
    Default: "Information"
    Override:
      SS14: "Information"
      Microsoft: "Warning"
      Microsoft.Hosting.Lifetime: "Information"
      Microsoft.AspNetCore: "Warning"
      Microsoft.AspNetCore.DataProtection: "Error"

  WriteTo:
    - Name: Console
      Args:
	OutputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3} {SourceContext}] {Message:lj}{NewLine}{Exception}"

  Enrich: [ "FromLogContext" ]

DetailedErrors: true
ConnectionStrings:
    default: "Server=postgres;Port=5432;Database=ss14_mapserver;User Id=ss14mapserver;Password=secret;"

AllowedHosts: "*"

Auth:
  ApiKey: "secret"

Github:
  AppName: "app-name"
  AppId: 0000
  AppPrivateKeyLocation: "/app/private-key.pem"
  AppWebhookSecret: "secret"

Git:
  RepositoryUrl: "https://github.com/space-syndicate/space-station-14.git"
```
</details>
