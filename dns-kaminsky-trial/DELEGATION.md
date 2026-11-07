# Local parent/delegation path

The resolver (10.10.0.2) bootstraps the test. parent at 10.10.0.5.
The parent delegates victim.test. to ns1.victim.test. (10.10.0.3),
and evil.test. to ns1.evil.test. (10.10.0.4).
This is a local parent bootstrap, not a complete private root hierarchy.

Files:
- docker-compose.yml: adds the parent service on the existing internal network.
- parent/Dockerfile: installs BIND and copies its configuration and test. zone.
- parent/named.conf: includes options and zone configuration.
- parent/named.conf.options: authoritative only; listens on 10.10.0.5.
- parent/named.conf.local: loads the test. primary zone.
- parent/db.test: contains the parent SOA, NS, child delegations, and glue.
- resolver/named.conf: includes options and local zones; removes the old root override.
- resolver/named.conf.local: replaces the victim.test stub with a test stub.

Run these commands in Ubuntu, from this project directory:

```sh
docker compose up -d --build
docker compose exec parent named-checkconf
docker compose exec parent named-checkzone -i local test /etc/bind/db.test
docker compose exec resolver named-checkconf
```

If a query immediately after startup fails, inspect `docker compose logs parent resolver`.
The resolver may need a moment to retrieve the parent's stub NS records.

## Prove the parent returns a referral

```sh
docker compose exec attacker dig @10.10.0.5 www.victim.test A +norecurse
```

Expected: NOERROR, no answer, AUTHORITY contains victim.test. NS
ns1.victim.test., ADDITIONAL contains ns1.victim.test. A 10.10.0.3.
The parent does not contain a www.victim.test address record.

## Prove the child is authoritative

```sh
docker compose exec attacker dig @10.10.0.3 www.victim.test A +norecurse
```

Expected: authoritative answer (aa flag), www.victim.test. A 10.10.0.20.

## Prove recursion through the delegation

First restart the resolver to clear its in-memory child cache:

```sh
docker compose restart resolver
```

Then, in one terminal, start the capture:

```sh
docker compose exec resolver tcpdump -ni any -nn -vv 'udp port 53 and (host 10.10.0.5 or host 10.10.0.3)'
```

In another terminal, query the resolver:

```sh
docker compose exec attacker dig @10.10.0.2 www.victim.test A
```

Expected answer: 10.10.0.20. On a cold child cache, capture should show
resolver -> parent, parent -> resolver referral, resolver -> child,
child -> resolver answer. Parent stub refresh traffic may also appear.

Repeat the query to observe caching; a warm cache may bypass the parent and child.

```sh
docker compose exec attacker dig @10.10.0.2 www.evil.test A
```

Expected: 10.10.0.4.

Do not put a victim.test stub, static-stub, primary zone, or forward-only override
back in the resolver; that can bypass or constrain delegation discovery. The test
parent is the only configured bootstrap here. This change establishes normal DNS
resolution; it does not disable defenses or implement the attack.

