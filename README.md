# Mattermost Docker config

A docker-compose stack to deploy on a Linux box.

## Requirements

See [openstack_setup.md](./openstack_setup.md) for requirements in an Openstack project

## Setup

Clone the repo, change to directory and run `docker compose up -d`.

It will fail because you need to add your `.env` file containing the env vars it's complaining about...

## Config

Mattermost Env Config can be found [here](https://docs.mattermost.com/administration-guide/configure/environment-configuration-settings.html)

## Backups

Are done automatically to the defined restic repository.

For more backup-docs look at [resticker](https://github.com/djmaze/resticker/tree/master).

## To Do

- Pre & Post Commands of Backup fail because the compose folder & command to execute is not properly escaped
- The docker volumes should live on a separete disk
- Cleanup and automate some things