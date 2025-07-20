---
layout: default
title: Installation with Docker
nav_order: 1
parent: User Guide
---

{: .warn }
If you just want to run Geneac locally for testing, you should be looking at [Developer Quickstart](development/quickstart) instead!

{: .info }
Geneac uses local file storage and a SQLite database. This makes it ideal for self-hosting, but means that it will not scale up to
multiple host machines by default. If this is a usecase you have, you can reach out to me and we can discuss it further.

## Installation with Docker

Docker's not just for experts!

Geneac is bundled up into a nice ~~little~~ container hosted on the GitHub Container Registry,
which you can pull yourself.

```bash
$ docker pull ghcr.io/mrysav/geneac:latest
```

You can run the container as-is, but there are some environment variables you'll want to configure first.

### Environment Variables

| Environment Variable | Suggested Value |
| -------------------- | --------------- |
| `REDIS_URL`          | URL to Redis container, eg. `redis://geneac-redis:6379` |
| `RAILS_FORCE_SSL`    | If you are hosting behind a reverse proxy with HTTPS, set this to `1` |
| `RAILS_ENV`          | `production` |
| `RAILS_LOG_TO_STDOUT`| `true` |
| `RAILS_SERVE_STATIC_FILES` | `true` |
| `SECRET_KEY_BASE`    | Generate a new one with `bin/rake secret` or similar random generator |

### Docker Compose Example

Here is an example `docker-compose.yml` file you can use to run Geneac. You can adjust the environment variables as needed.

```yaml
services:
  web:
    image: ghcr.io/mrysav/geneac:latest
    command: ./bin/thrust ./bin/rails server
    restart: unless-stopped
    ports:
      - '3000:3000'
    environment:
      - RAILS_ENV=production
      # Uncomment if needed
      # - RAILS_FORCE_SSL=1
      - RAILS_LOG_TO_STDOUT=true
      - RAILS_SERVE_STATIC_FILES=true
      - REDIS_URL=redis://geneac-redis:6379
      # Make sure to generate your own!
      - SECRET_KEY_BASE=
    volumes:
      - geneac-data:/rails/storage/

  worker:
    image: ghcr.io/mrysav/geneac:latest
    command: ./bin/rake resque:work
    restart: unless-stopped
    environment:
      - QUEUE=*
      - RAILS_ENV=production
      # - RAILS_FORCE_SSL=1
      - RAILS_LOG_TO_STDOUT=true
      - RAILS_SERVE_STATIC_FILES=true
      - REDIS_URL=redis://redis:6379
      # This should be the same one as above!
      - SECRET_KEY_BASE=
    volumes:
      - geneac-data:/rails/storage/

  redis:
    image: docker.io/library/redis:8-alpine
    command: redis-server --save 60 1 --loglevel warning
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli","ping"]

volumes:
  geneac-data:
```

## Post-Installation

When you start Geneac, you will be presented with the home page.

![Homepage of Geneac](image.png)

To get an administrator account, you must follow these steps:

1. Register an account by clicking the "Sign in or register" link and filling out the "Sign up" form
1. In your terminal window, you must grant administrator access to the account you just created:
    ```bash
    $ user@server:~$ docker compose run web bash
    $ rails@32f8366bfb15:/rails$ bin/rails admin:grant[your-email-here@your-domain.com]
    ```

The account you just created will now have administrator permissions.
