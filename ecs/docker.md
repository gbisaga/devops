### Building docker image
- Simplest Dockerfile
```
FROM busybox
CMD echo "hi"
```
- docker build -t container-name .
- ENTRYPOINT vs CMD - ENTRYPOINT has args, cannot be overridden
- RUN - adds a new layer with results of the RUN command
- EXPOSE <port> - port(s)
- Multistage builds
  - use more than one FROM - all except last are `FROM <img> AS <name>`
  - `CMD copy --from=<name> /from/dir /to/dir`

https://hub.docker.com/r/gkoenig/simplehttp
docker.io/gkoenig/simplehttp:latest
port 8000
