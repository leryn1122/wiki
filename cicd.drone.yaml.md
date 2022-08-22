```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: build-on-commit-nightly
    image: plugins/docker
    settings:
      registry: docker.leryn.top
      repo: docker.leryn.top/leryn/image-name
      tags: [nightly]
    volumes:
      - name: docker-socket
        path: /var/run/docker.sock
    when:
      branch:
        exclude: [master, feature/*, stable]
      event:
        include: [cron, push, pull_request]
  - name: build-on-commit-latest
    image: plugins/docker
    settings:
      registry: docker.leryn.top
      repo: docker.leryn.top/leryn/image-name
      tags: latest
    volumes:
      - name: docker-socket
        path: /var/run/docker.sock
    when:
      branch:
        include: [master, feature/*, stable]
      event:
        include: [push, pull_request]
  - name: build-on-tag
    image: plugins/docker
    settings:
      registry: docker.leryn.top
      repo: docker.leryn.top/leryn/image-name
      tags: stable
    volumes:
      - name: docker-socket
        path: /var/run/docker.sock
    when:
      event:
        include: tag
        exclude: [cron, push, pull_request]
volumes:
  - name: docker-socket
    host:
      path: /var/run/docker.sock

```
