# build-wagtail

An opinionated tool for quickly deploying a Wagtail project by turning it into a local Docker contaier. Run it in a Wagtail CMS directory and it generates a Docker image and a compose file in place ready for production run. It sets up the high-performance [granian](https://github.com/emmett-framework/granian) Python server instead of uwsgi and optionally also configures Caddy.

Needs Docker with the compose plugin to be installed on the host.

## Install

```shell
uv tool install build-wagtail   # or pipx install build-wagtail
```

## Usage

`cd` to the Wagtail project dir (where `manage.py` is) and:

```shell
build-wagtail example.org,www.example.org

# That generates the Docker file and a compose.yml file,
# and runs Docker build locally to prepare the image.

# Then, to serve the site.
docker compose up -d
docker compose exec web python manage.py createsuperuser
```

The first hostname is canonical. The site runs on `127.0.0.1:8000`. granian serves the app as well as `/static` and `/media` With `--caddy`, a Caddy container in front serves everything including TLS.

| Option            | Default                                         |                                                    |
|-------------------|-------------------------------------------------|----------------------------------------------------|
| `--port`          | `8000`                                          | localhost port the site is served on               |
| `--caddy`         | off                                             | Run Caddy in a container                           |
| `--caddy-image`   | `caddy:2.10-alpine`                             | Caddy image to use with `--caddy`                  |
| `--workers`       | `3`                                             | Number of granian server workers                   |
| `--threads`       | `4`                                             | granian threads per worker that run the app        |
| `--python`        | `.python-version`, else `3.13`                  | Python version of the image                        |
| `--settings`      | from `manage.py`, or `<project>.settings.production` | Settings module to use                        |
| `--no-build`      | off                                             | Write the files but skip `docker compose build`    |


## How it works

The tool simply generates opinionated configuration files and optionally runs `docker build` to create the image locally.

The image only has Python, the project's dependencies and granian. The project directory itself is mounted into the container, so the site runs from the files in the directory. After changing code, run `docker compose restart web`. After changing dependencies, re-run `build-wagtail`.

The tool generates:

- `.env`: secret key, hostnames and the container user, which is the project directory's owner. If it already exists, it's not overwritten.
- `.wagtail-docker/` and `compose.yml` (regenerated on every run).
