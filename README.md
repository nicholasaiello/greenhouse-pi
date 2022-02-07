# Monitoring Tools for a Pimoroni Enviro+

> This repository is inspired by [geerlingguy/internet-monitoring](https://github.com/geerlingguy/internet-monitoring) and [tijmenvandenbrink/enviroplus_exporter#getting-started](https://github.com/tijmenvandenbrink/enviroplus_exporter#getting-started). It has only been tested on a Raspberry Pi 4 running Pi OS 64-bit (Server), and a Raspberry Pi Zero 2 running Pi OS 64-bit (Client).

Stand-up a Docker [Prometheus](http://prometheus.io/) stack containing Prometheus, Grafana with [blackbox-exporter](https://github.com/prometheus/blackbox_exporter) to collect and graph home Internet reliability and throughput.

## Pre-requisites

Make sure Docker and [Docker Compose](https://docs.docker.com/compose/install/) are installed on your Docker host machine.

### Install
Some aspects of these steps may be dated, but this has typically worked for me.

Run `docker` install script
```sh
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -
curl -fsSL https://get.docker.com | bash
```

Add `docker` to User's groups
```sh
sudo usermod -aG docker $(whoami)
```

Install `docker-compose`
```sh
pip3 install docker-compose
```

Config for docker daemon
```sh
# this may require you to run `sudo su` first
if ! [ -f "/etc/docker/daemon.json" ]; then
    echo '
    {
      "experimental": true,
      "live-restore": true,
      "ipv6": false,
      "icc": false,
      "no-new-privileges": false
    }' > /etc/docker/daemon.json
fi
```

Enable docker service
```sh
sudo systemctl enable docker
sudo systemctl enable docker
```

## Quick Start

### Server
Update the IP for your target greenhouse by defining an environment variable (`GREENHOUSE_HOST`), or by hardcoding the value in `docker-compose.yml`
```yaml
    ...
    extra_hosts:
      - 'greenhouse-pi.local:${GREENHOUSE_HOST:-192.168.0.163}'
    ...
```

Builds the entire Grafana and Prometheus stack automagically.
```
git clone https://github.com/nicholasaiello/greenhouse-pi
cd greenhouse-pi
docker-compose up -d
```

The Grafana Dashboard is now accessible via: `http://<Host IP Address>:3030` for example http://localhost:3030

username - admin
password - wonka (Password is stored in the `config.monitoring` env file)

The DataSource and Dashboard for Grafana are automatically provisioned.

### Client

Use the following project on your `target` Pi, the one you plan to pull metrics from. 
> https://github.com/tijmenvandenbrink/enviroplus_exporter#getting-started


Once setup, you can verify that Prometheus is correctly pulling metrics from your target, by visiting: http://localhost:9090/targets

## Interesting urls

http://localhost:9090/targets shows status of monitored targets as seen from prometheus - in this case which hosts being pinged and speedtest. note: speedtest will take a while before it shows as UP as it takes about 30s to respond.

http://localhost:9090/graph?g0.expr=probe_http_status_code&g0.tab=1 shows prometheus value for `probe_http_status_code` for each host. You can edit/play with additional values. Useful to check everything is okey in prometheus (in case Grafana is not showing the data you expect).

http://localhost:9115 blackbox exporter endpoint. Lets you see what have failed/succeded.

## Thanks and a disclaimer

Thanks to @geerlingguy and @tijmenvandenbrink for their original projects, of which this was based on.

This setup is not secured in any way, so please only use on non-public networks, or find a way to secure it on your own.
