Traefik 1.x and 2.x
===================

We recently finished a migration from [Traefik](https://traefik.io/)
v1 to v2 and received some questions about the process.

If you're upgrading be sure to read over the [migration docs](https://doc.traefik.io/traefik/migration/v1-to-v2/) too.

## Notes before Starting

I've included a self-signed SSL certificate and private key for ease of use.
Don't use them in production.

I use a separate Docker network for each version of Traefik for better
separation, but it's not strictly required.

The configurations here are stripped of most extraneous configurations.
When running in production be sure you secure all of the admin endpoints,
including your metrics collections in whatever way works for you.
## Prerequsites

To follow this guide as is you'll need a few things:

- cURL
- [Docker](https://docs.docker.com/get-docker/) running in Swarm mode (outside the scope of this document).
- Load Balancing Networks Created:
```bash
docker network create --driver=overlay traefik_v1
docker network create --driver=overlay traefik_v2
```

## Running

**Warning:** Copy and pasting the commands below may break things if you run it on a swarm you care about. If you're not sure, you should probably use an empty
swarm cluster.

To start v1 run:

```bash
docker stack deploy \
--with-registry-auth \
-c traefik-v1.yaml \
traefik-v1
```

To start v2 run:

```bash
docker stack deploy \
--with-registry-auth \
-c traefik-v2.yaml \
traefik-v2
```

Once things are running we'll end up with a few things running:
- Traefik v1
  - HTTPS listening on port 8443
  - [Dashboard](https://localhost:8080/dashboard/) listening on port 8080
- Traefik v2
  - HTTPS listening on port 9443
  - [Dashboard](https://localhost:8000/dashboard/) listening on port 8000
- whoami_v1
  - Listening internally, accessible only through Traefik v1
- whoami_v2
  - Listening internally, accessible only through Traefik v2

### Testing Traefik v1

You should get back some metadata from the whoami_v1 container from

`curl -k -H "Host: test-v1" https://localhost:8443/`

note that changing `test-v1` to anything else will result in a 404 error.

### Testing Traefik v2

You should get back some metadata from the whoami_v2 container from

`curl -k -H "Host: test-v2" https://localhost:9443/`

Note that changing `test-v2` to anything else will result in a 404 error.

### Running in Both Stacks

In some scenarios you can serve an application from both stacks at the
same time.

Deploy our dual stack application with
```bash
docker stack deploy \
--with-registry-auth \
-c dualstack-app.yaml \
dualstack-app
```

Wait about 10 seconds for both instances of Traefik to pick up our new
application, then test.

To test Traefik v1 we'll use

`curl -k -H "Host: test-dual" https://localhost:8443/`


Traefik v2 is requires changing the port so that we run

`curl -k -H "Host: test-dual" https://localhost:9443/`

#### Notes about Multiple Versions

The multiple version setup shown here makes a lot of assumptions about
networks, so it may not work in all environments, but it should be a start.

## Tearing Down

When you're all done you can remove everything we created with
```bash
docker stack rm docker stack rm dualstack-app traefik-v1 traefik-v2
docker network rm traefik_v1 traefik_v2
```