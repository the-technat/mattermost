# Mattermost Docker config

A docker-compose stack to deploy on a Linux box.

## Requirements

See [openstack_setup.md](./openstack_setup.md) for requirements and docs on how to use this config in an Openstack project

## Setup

Clone the repo, change to directory and run `docker compose up -d`.

It will fail because you need to add your `.env` file containing the env vars it's complaining about...

Add a `.env` file containing:

```
DOMAIN=TODO
POSTGRES_USER=mattermost
POSTGRES_PASSWORD=TODO
POSTGRES_DB=mattermost
RESTIC_REPOSITORY="swift:mattermost_backup:/"
RESTIC_PASSWORD=TODO
OS_AUTH_URL=TOOD
OS_APPLICATION_CREDENTIAL_SECRET=TODO
OS_APPLICATION_CREDENTIAL_ID=TODO
```

Enter the values that you generated. Also ensure the domain you enter there is the same as in the [Caddyfile](./Caddyfile) so that Caddy will request a certificate for the correct domain.

## Config

Mattermost Env Config can be found [here](https://docs.mattermost.com/administration-guide/configure/environment-configuration-settings.html)

## Backups

Are done automatically to the defined restic repository.

For more backup-docs look at [resticker](https://github.com/djmaze/resticker/tree/master).
