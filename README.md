# PostgreSQL development environment management script

## Configuration settings

### Setting for make

| Config | Defaults | Description |
| --- | --- | --- |
| MAKEFLAGS | | make settings ex: '-j 4'|

### Setting for pg-env 

| Config | Defaults | Description |
| --- | --- | --- |
| PGV | None | PostgreSQL Major Version or HEAD |
| GIT_DIR | ${HOME}/gitwork | Your GIT workspace directory name |
| PG_DEV_NAME | postgresql-<PGV> | Name of the source code folder (a git worktree) |
| PG_INST_NAME | <PG_DEV_NAME>-INST | Name of the install folder |
| PG_ENV_CFLAGS | | Extra CFLAGS |

PostgreSQL extra configs shall be stored in `${GIT_DIR}/pgconfigs/` in this naming convention:
```
pg${PGV}.conf
pg${PGV}-<Cluster #>.conf
```

PostgreSQL local configs per folder are named similarly but starts with (.) dot and stored in worktree folder. These will overlay on top of `${GIT_DIR}/pgconfigs/` settings.

```
.pg${PGV}.conf
.pg${PGV}-<Cluster #>.conf
```

### Advance setting for pg-env 

| Config | Defaults | Description |
| --- | --- | --- |
| PG_ENV_DEBUG | | Set this var to non empty to print some debug info |

## Setting up environment

You can create ~/.pg-env with your global settings

You can create ./.pg-env with your per folder settings

You can also export PGV=<major version|HEAD>

PGPORT will be 5400 + PGV. When PGV is set to HEAD PGPORT is set to 5432
When working with multiple Clusters, passed on id is added as a prefix to PGPORT
Note: Max 1 - 5 Clusters are allowed.

## Setup

- Clone  
- Symlink `ln -s <Clone Dir>/pg-env ~/bin/pg-env`  

## Setup commitfest Patch testing

Visit https://commitfest.postgresql.org/ and click on active commitfest and from the URL record CommitFest ID
Now click on a commitfest entry and from the URL record commitfest Entry ID
Ex: https://commitfest.postgresql.org/48/4962/  commitfest IS is 48 and Entry ID is 4962

chdir to folder where PostgreSQL source is cloned

For testing against development HEAD

`pg-env commitfest 48 4962`

For testing against a stable branch ex: release 12

`pg-env commitfest 48 4962 REL_12_STABLE`

## Activate PostgreSQL Development Environment
`source ~/bin/pg-env`  

## Run distclean on PostgreSQL source
`distclean`  

## Compile PostgreSQL source and install
`compile`  

## Compile PostgreSQL contrib and install
`compile_contrib`  

## Start PostgreSQL
`startdb <Clusters>`  
ex:  
  startdb  
  startdb 1 2 3  

## Stop PostgtreSQL
`stopdb <Clusters>`  
ex:  
  stopdb  
  stopdb 1 2 3  

## Restart PostgreSQL
`restartdb <Clusters>`  
ex:  
  restartdb  
  restartdb 1 2 3  

## Remove PGDATA
`cleandb <Clusters>`  
ex:  
  cleandb  
  cleandb 1 2 3  

## Initialize PostgreSQL Cluster and include custom config
`setupdb <Clusters>`  
ex:  
  setupdb  
  setupdb 1 2 3  

## Reset DB
`resetdb <Clusters>`  
ex:  
  resetdb  
  resetdb 1 2 3  

## Reset database and install direcotory
`resetall <Clusters>`  
ex:  
  resetall  
  resetall 1 2 3  

## deactivate the environment
`deactivate`  
Exit from pg-env  

