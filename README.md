# PostgreSQL development environment management script

## Setting up environment

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

