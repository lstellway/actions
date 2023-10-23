# Deploy docker-compose SSH

GitHub [composite action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) for deploying a set of docker-compose files to a remote environment via SSH.

## Inputs

This action utilizes two external GitHub actions _(and supports all of their inputs)_:

-   [webfactory/ssh-agent](https://github.com/webfactory/ssh-agent) _([input variables](https://github.com/webfactory/ssh-agent/tree/v0.8.0#action-inputs))_
    -   Used to prepare the SSH environment
-   [docker/login-action](https://github.com/docker/login-action) _([input variables](https://github.com/docker/login-action/tree/v3#inputs))_
    -   Used to facilitate logging into the Docker registry

Additionally, the following inputs can be used in the [`steps.with`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepswith) object hashmap:<br />
_(required fields marked with `*`)_

| Name                     | Type   | Default | Description                                                                                                                      |
| ------------------------ | ------ | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `ssh-host`\*             | String |         | The remote hostname or IP address of the Docker host                                                                             |
| `ssh-persist`            | String | `5m`    | The amount of time SSH sessions should persist _([more info](https://docs.docker.com/engine/security/protect-access/#ssh-tips))_ |
| `ssh-port`               | Number | `22`    | Public port on which to access SSH                                                                                               |
| `ssh-user`\*             | String |         | SSH user used for the remote SSH session                                                                                         |
| `docker-compose-files`\* | String |         | Docker compose files to deploy _(separated by comma or newline)_                                                                 |
| `docker-project`         | String |         | The Docker-compose project name _([documentation](https://docs.docker.com/compose/reference/#use--p-to-specify-a-project-name))_ |

## Usage

Example using this action to deploy a set of Docker compose files to a remote environment:

```yml
name: docker-compose-deploy

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Deploy Docker compose files
        uses: lstellway/actions/deploy-docker-compose-ssh@0.0.1
        with:
          # SSH
          ssh-user: ${{ secrets.SSH_USER }}
          ssh-host: ${{ secrets.SSH_HOST }}
          ssh-port: ${{ secrets.SSH_PORT }}
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
          log-public-key: true
          # Docker
          # Multiple compose files can be provided, separated by a comma or newline
          docker-compose-files: docker-compose.yml,docker-compose.dev.yml
          docker-project: myprojectname
          docker-registry: ${{ secrets.DOCKER_REGISTRY }}
          docker-username: ${{ github.repository_owner }}
          docker-password: ${{ secrets.DOCKER_TOKEN }}
          docker-logout: true
        env:
          CONTAINER_RUNTIME_VARIABLE: ${{ secrets.CONTAINER_RUNTIME_VARIABLE }}
```