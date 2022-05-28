# frp on fly.io
[Fast reverse proxy](https://github.com/fatedier/frp) on [fly.io](https://fly.io)

Run your own frp tunnel for free (within free tier) on fly.io

Now you can have ngrok TCP/UDP tunnel with the ports you want, not randomly generated ports on ngrok unless you [pay for the pro monthly](https://ngrok.com/pricing).

## fly.io Deployment
You need [flyctl](https://github.com/superfly/flyctl)

1. Clone this repository.
2. Create an app on fly.io `fly launch --copy-config --name app-name`.
3. To deploy, type y if prompted to deploy now.
4. Set environment variables `fly secrets set -a app-name FRP_TOKEN=12345678 FRP_DASH_USER=admin FRP_DASH_PWD=admin`.
5. Try to connect to frps using `server_addr = app-name.fly.dev`, `server_port = 7000`, `protocol = kcp`, and `token = 12345678` in frpc.ini.

Don't forget to change the token so that others can't use your frp tunnel.

You can also view https://app-name.fly.dev in browser to view the frps dashboard.

## Change server configuration
Type `fly deploy -a app-name` on the repository after editing frps.ini

## Switch
fly.io runs app 24/7, if you are not using your tunnel for a while, it is recommended to suspend it to conserve free tier and resources.

Suspend frp `fly suspend app-name`\
Resume frp `fly resume app-name`

## TCP or UDP tunnel, not both
Since in fly.io, it is [required to bind to `fly-global-services`](https://fly.io/docs/app-guides/udp-and-tcp/) in order for UDP to work, but frp `proxy_bind_addr` only allow to bind in one address, so we need to disable TCP if you want to use UDP as TCP does not work on `fly-global-services`.

You need to have a separate frp instance if you need to tunnel both TCP and UDP. One for TCP using `proxy_bind_addr = 0.0.0.0` and one for UDP using `proxy_bind_addr = fly-global-services`.

## KCP Protocol
[KCP](https://github.com/skywind3000/kcp/blob/master/README.en.md) (a protocol built on UDP) is used by default so that a TCP meltdown (TCP over TCP tunnel) will not happen and to reduce latency (like for game servers).

You can also use TCP if KCP is not working for you by uncommenting and commenting the lines in frps.ini and fly.toml

## Example frpc.ini
```
[common]
server_addr = app-name.fly.dev
server_port = 7000
protocol = kcp
token = 12345678

# TCP tunnel, requires proxy_bind_addr = 0.0.0.0 in frps.ini
[minecraft]
type = tcp
local_ip = 127.0.0.1
local_port = 25565
remote_port = 25565

# UDP tunnel, requires proxy_bind_addr = fly-global-services in frps.ini
[torrent-tracker]
type = udp
local_ip = 127.0.0.1
local_port = 25565
remote_port = 25565
```

### fly.io free tier
fly.io requires a credit card in order to work, if you don't have a credit card or if you are afraid that fly.io will charge you so much, it is recommend to buy prepaid credits that can be used with virtual credit cards.

### HTTP Tunneling
If you are tunneling HTTP apps instead of TCP/UDP, I recommend to just use [Cloudflare Tunnel](https://www.cloudflare.com/products/tunnel/).\
You can also tunnel HTTP apps on this frp by using a custom port like 8080.\
If you need to use standard 80 and 443 port, you need to disable the frps dashboard.

### More infos
[anderspitman/awesome-tunneling](https://github.com/anderspitman/awesome-tunneling)

# 🖕 Carrier-grade Network Address Translation (CGNAT)