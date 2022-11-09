# Rathole short setup

[Rathole](https://github.com/rapiz1/rathole) can help to expose the service on the device behind the NAT to the Internet, via a server with a public IP.

This set of scripts simplifies first setup of Rathole reverse proxy tunnel.

## Features

- Downloads and installs Rathole binary and systemd services.
- Creates config files for first tunnel.
- Tunnel transfer will protected with encription by [Noise Protocol](https://noiseprotocol.org/noise.html).


## Requirements

- Linux server with public IP.
- SSH access to server with superuser privileges (sudo).
- Optionally you need gateway webserver like Caddy, Nginx, Traefik etc.
- Optionally you need domain name attached to server IP (Do you need detailed instruction?).


## Installation steps

### 1. Preparation

First, download or clone this repository to your local machine.

```
$ git clone git@github.com:devgkz/first-rathole-setup.git
$ cd first-rathole-setup
```

### 2. Install binaries

Then upload `install-bin` script to server.

```
$ scp ./install-bin user@server.com:~
```

Connect to server by ssh and run script. Make it executable before. System may request sudo password.

```
$ chmod +x install-bin && ./install-bin
```

### 3. Define configuration

After successful installation generate keypair on server for Noise Protocol.
```
$ rathole --genkey
```

Locally open in text editor files `install-server` and `install-client`, and edit config-block:

- Set private and public keys generated above. 
- Set address that server is listen for tunnel connection.
- Set your service id and token equal both on server and client.
- And addresses listens on both sides of tunnel.

### 4. Install to server

After editing upload `install-server` file to server

```
$ scp ./install-server user@server.com:~
```

Connect to server by ssh and run script. Make it executable before. System may request sudo password.

```
$ chmod +x install-server && ./install-server
```

After installation service rathole@default will started automatically.

### 5. Gateway configuration (optional)

If you need additinal gateway webserver then configure it. I prefer [Caddy](https://caddyserver.com/v2) as it obtains and renews TLS certificates for your sites automatically.

For example, install and configure Caddy for my tunnel and domain `example.server.com` on Ubuntu server.

```
$ sudo apt install caddy
$ sudo nano /etc/caddy/Caddyfile
```

Add to `Caddyfile` following block:

```
example.server.com {
	reverse_proxy localhost:4201
}
```

Then restart Caddy.

```
$ sudo systemctl restart caddy
```

Now Caddy receive requests to site `https://example.server.com` and translate they to the rathole tunnel port 4201. 

### 6. Install to client

Let's configure client side of tunnel. Back to local machine in project directory (first-rathole-setup) and run:

```
$ chmod +x install-client & ./install-client
```

After installation service ratholec@default will started automatically.

Start your local service and access it by public IP:port or domain name.


## Troubleshooting

You may need open input port on server if firewall is on. On Ubuntu it may be made via `ufw`.

```
$ sudo ufw allow 2400
```

Check the systemd service rathole is running on server

```
$ sudo systemctl stop rathole@default
```

Check the systemd service ratholec is running on client

```
$ sudo systemctl stop ratholec@default
```

## Links

Original Rathole project https://github.com/rapiz1/rathole

Noise Protocol specification https://noiseprotocol.org/noise.html

