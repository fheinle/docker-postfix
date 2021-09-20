# Mail Relay for Docker Containers

This is a simple container running postfix. It will accept mails and relay them over an authenticated smarthost to their destination.

Mails will be accepted on Port 25 for delivery without authentication by the containers. Access control is meant to happen on the network level, i.e. by limiting which hosts/containers can connect to the relay and whitelisting network addresses.

## Building

Building the container will fetch `debian:bullseye-slim` as the container's base.

```shell
$ docker-compose build
```

## Configuration

Edit `env.sample` and save it as `env`.

* `mydomain`: Domain part of outgoing mail addresses
* `relayhost`: fqdn for your relay host
* `relayuser`: username for authentication at the remote mail relay
* `relaypassword`: password for authentication at the remote mail relay


Optional configuration:

* `myhostname`: Hostname the docker container should assume for itself. Defaults to `mail`.
* `mynetworks`: Networks (CIDR) that are whitelisted for mail delivery. Defaults to `192.168.0.0/16,172.16.0.0/12`.
* `relayport`: Port the remote mail relay is reachable at. Defaults to `587` (submission)

## Running

Start the container:

```shell
$ docker-compose up -d
```

Watch logfiles:

```shell
$ docker-compose logs -f
```

## App configuration

Make sure your apps' `docker-compose.yaml` have the `mail` network configured for all services that should be able to send mails, e.g.:


```yaml
---
version: "3.5"
services:
  busybox:
    image: busybox
  networks:
    - mail
networks:
  mail:
    external: true
    name: mail
```

Pay attention to the `networks` settings (both of them).

Then, configure your apps to the following settings:

* SMTP Host: `mail_postfix_1`
* SMTP Port: `25`
* SMTP Auth: none at all
* StartTLS/SSL: Only `StartTLS` will work on port 25. It's possible, but not required when all containers run on the same machine and communication doesn't go over the wire.
* Certificate validation: `off`. We won't generate valid certificates for `mail_postfix_1`.

## Copyright

Florian Heinle <github@florianheinle.de>

MIT-License (see `LICENSE`)
