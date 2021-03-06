# frp

[![Build Status](https://travis-ci.org/fatedier/frp.svg)](https://travis-ci.org/fatedier/frp)

[README](README.md) | [中文文档](README_zh.md)

## What is frp?

frp is a fast reverse proxy to help you expose a local server behind a NAT or firewall to the internet.Now, it supports tcp, http and https protocol when requests can be forwarded by domains to backward web services.

## Catalog

* [What can I do with frp?](#what-can-i-do-with-frp)
* [Status](#status)
* [Architecture](#architecture)
* [Example Usage](#example-usage) 
  * [Communicate with your computer in LAN by SSH](#communicate-with-your-computer-in-lan-by-ssh)
  * [Visit your web service in LAN by specific domain](#visit-your-web-service-in-lan-by-specific-domain)
* [Features](#features)
  * [Authentication](#authentication)
  * [Encryption and Compression](#encryption-and-compression)
  * [Reload configures without frps stopped](#reload-configures-without-frps-stopped)
  * [Privilege Mode](#privilege-mode)
* [Development Plan](#development-plan)
* [Contributing](#contributing)
* [Contributors](#contributors)

## What can I do with frp?

* Expose any http and https service behind a NAT or firewall to the internet by a server with public IP address(Name-based Virtual Host Support).
* Expose any tcp service behind a NAT or firewall to the internet by a server with public IP address.
* Inspect all http requests/responses that are transmitted over the tunnel(future).

## Status

frp is under development and you can try it with latest release version.Master branch for releasing stable version when dev branch for developing.

**We may change any protocol and can't promise backward compatible.Please note the release log when upgrading.**

## Architecture

![architecture](/doc/pic/architecture.png)

## Example Usage

First, download the latest version programs from [Release](https://github.com/fatedier/frp/releases) page according to your os and arch.

Put **frps** and **frps.ini** to your server with public IP.

Put **frpc** and **frpc.ini** to your server in LAN.

### Communicate with your computer in LAN by SSH

1. Modify frps.ini, configure a reverse proxy named [ssh]:

  ```ini
  # frps.ini
  [common]
  bind_port = 7000

  [ssh]
  listen_port = 6000
  auth_token = 123
  ```

2. Start frps:

  `./frps -c ./frps.ini`

3. Modify frpc.ini, set remote frps's server IP as x.x.x.x:

  ```ini
  # frpc.ini
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  auth_token = 123

  [ssh]
  local_port = 22
  ```

4. Start frpc:

  `./frpc -c ./frpc.ini`

5. Connect to server in LAN by ssh assuming that username is test:

  `ssh -oPort=6000 test@x.x.x.x`

### Visit your web service in LAN by specific domain

Sometimes we need to expose a local web service behind a NAT network to others for testing with your own domain and unfortunately we can't resolve a domain to a local ip.

Howerver, we can expose a http or https service using frp.

1. Modify frps.ini, configure a http reverse proxy named [web] and set http port as 8080, custom domain as www.yourdomain.com:

  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  vhost_http_port = 8080

  [web]
  type = http
  custom_domains = www.yourdomain.com
  auth_token = 123
  ```

2. Start frps:

  `./frps -c ./frps.ini`

3. Modify frpc.ini and set remote frps server's IP as x.x.x.x. The local_port is the port of your web service:

  ```ini
  # frpc.ini
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  auth_token = 123

  [web]
  type = http
  local_port = 80
  ```

4. Start frpc:

  `./frpc -c ./frpc.ini`

5. Resolve A record of www.yourdomain.com to x.x.x.x or CNAME record to your origin domain.

6. Now your can visit your local web service from url `http://www.yourdomain.com:8080`.

## Features

### Authentication

`auth_token` is used in frps.ini for authentication when frpc login in and you should configure it for each proxy.

Client should set a global `auth_token` equals to frps.ini.

Note that time duration bewtween frpc and frps shouldn't exceed 15 minutes because timestamp is used for authentication.

### Encryption and Compression

Defalut value is false, you could decide if the proxy should use encryption or compression whether the type is:

```ini
# frpc.ini
[ssh]
type = tcp
listen_port = 6000
auth_token = 123
use_encryption = true
use_gzip = true
```

### Reload configures without frps stopped

If your want to add a new reverse proxy and avoid restarting frps, you can use this feature.

1. `dashboard_port` should be set in frps.ini:

  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  dashboard_port = 7500
  ```

2. Start frps:

  `./frps -c ./frps.ini`

3. Modify frps.ini to add a new proxy [new_ssh]:

  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  dashboard_port = 7500

  [new_ssh]
  listen_port = 6001
  auth_token = 123
  ```

4. Execute `reload` command:

  `./frps -c ./frps.ini --reload`

5. Start frpc and [new_ssh] is available now.

### Privilege Mode

Privilege mode is used for who don't want to do operations in frps everytime adding a new proxy.

All proxies's configures are set in frpc.ini when privilege mode is enabled.

1. Enable privilege mode and set `privilege_token`.Client with the same `privilege_token` can create proxy automaticly:

  ```ini
  # frps.ini
  [common]
  bind_port = 7000
  privilege_mode = true
  privilege_token = 1234
  ```

2. Start frps:

  `./frps -c ./frps.ini`

3. Enable privilege mode for proxy [ssh]:

  ```ini
  # frpc.ini
  [common]
  server_addr = x.x.x.x
  server_port = 7000
  privilege_token = 1234

  [ssh]
  privilege_mode = true
  local_port = 22
  remote_port = 6000
  ```

4. Start frpc:

  `./frpc -c ./frpc.ini`

5. Connect to server in LAN by ssh assuming that username is test:

  `ssh -oPort=6000 test@x.x.x.x`

## Development Plan

* Dashboard page.
* Statistics and prestentation of traffic and connection info, etc.
* Support udp protocol.
* Connection pool.
* White list for opening specific ports in privilege mode.
* Support wildcard domain name.
* Url router.
* Load balance to different service in frpc.
* Debug mode for frpc, prestent proxy status in terminal.
* Inspect all http requests/responses that are transmitted over the tunnel.
* P2p communicate by make udp hole to penetrate NAT.

## Contributing

Interested in getting involved? We would like to help you!

* Take a look at our [issues list](https://github.com/fatedier/frp/issues) and consider submitting a patch
* If you have some wanderful ideas, send email to fatedier@gmail.com.

**Note: We prefer you to give your advise in [issues](https://github.com/fatedier/frp/issues), so others with a same question can search it quickly and we don't need to answer them repeatly.**

## Contributors

* [fatedier](https://github.com/fatedier)
* [Hurricanezwf](https://github.com/Hurricanezwf)
* [vashstorm](https://github.com/vashstorm)
* [maodanp](https://github.com/maodanp)
