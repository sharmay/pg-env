# PostgreSQL development environment management script

## Configuration settings

| Config | Defaults | Description |
| PGV | None | PostgreSQL Major Version or HEAD |
| GIT_DIR | ${HOME}/gitwork | Your GIT workspace directory name |
| PG_DEV_NAME | postgresql-<PGV> | Name of the source code folder (a git worktree) |
| PG_INST_NAME | <PG_DEV_NAME>-INST | Name of the install folder |

## Setting up environment

You can create ~/.pg-env with your global settings

You can create ./.pg-env with your per folder settings

export PGV=<major version|HEAD>

## Activate PostgreSQL Developmnent ENvironment
`source pg-env`

## Run distclean on PostgreSQL source
`distclean`

## Compile PostgreSQL source and install
`compile`

## Compile PostgreSQL contrib and install
`compile_contrib`

## Start PostgreSQL
`startdb <Nodes>`
ex:
  startdb
  startdb 1 2 3

## Stop PostgtreSQL
`stopdb <Nodes>`
ex:
  stopdb
  stopdb 1 2 3

## Restart PostgreSQL
`restartdb <Nodes>`
ex:
  restartdb
  restartdb 1 2 3

## Remove PGDATA
`cleandb <Nodes>`
ex:
  cleandb
  cleandb 1 2 3

## Initialize PostgreSQL Cluster and include custom config
`setupdb <Nodes>`
ex:
  setupdb
  setupdb 1 2 3

## Reset DB
`resetdb <Nodes>`
ex:
  resetdb
  resetdb 1 2 3

## Reset database and install direcotory
`resetall <Nodes>`
ex:
  resetall
  resetall 1 2 3

## deactivate the environment
`deactivate`
Exit from pg-env

