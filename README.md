# Concourse Docker-in-Docker

Docker in Docker Ubuntu container for Concourse CI. Optimized for use with [Concourse CI](http://concourse.ci/).    
Ubuntu image published at [fhivemind/concourse-dind](https://hub.docker.com/r/fhivemind/concourse-dind/), which includes Docker, Docker Compose, and Docker Squash.

Based on [karlkfi/concourse-dcind](https://github.com/karlkfi/concourse-dcind/).

## Features

- Automatically starts docker.
- Uses **errexit**, **pipefail**, and **nounset**.
- Configures timeout (`DOCKERD_TIMEOUT`) on dockerd start to account for misconfiguration (docker log will be output).
- Accepts arbitrary dockerd arguments via optional `DOCKER_OPTS` environment variable.
- Passes through `--garden-mtu` from the parent Gardian container if `--mtu` is not specified in `DOCKER_OPTS`.
- Sets `--data-root /scratch/docker` to bypass the graph filesystem if `--data-root` is not specified in `DOCKER_OPTS`.

## Run & Build
The image is available on DockerHub, but if you would like to build locally, clone and run

```bash
docker run fhivemind/concourse-dind
```

## Example
Here is an example of a Concourse [job](http://concourse.ci/concepts.html) that uses `fhivemind/concourse-dind` image to run a several containers in a task, and then runs the integration test suite.

```yaml
jobs:
- name: integration
  plan:
  - get: code
    params:
      depth: 1
    passed:
    - unit-tests
    trigger: true
  - task: integration-tests
    privileged: true
    config:
      platform: linux
      image_resource:
        type: docker-image
        source:
          repository: fhivemind/concourse-dind
      inputs:
      - name: code
      run:
        path: entrypoint.sh
        args:
        - bash
        - -ceux
        - |
          docker-compose -f code/example/integration.yml run tests
          docker-compose -f code/example/integration.yml down
          docker volume rm $(docker volume ls -q)
```
