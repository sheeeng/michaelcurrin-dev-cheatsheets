# Run containers
> How to use `run`, `start` and `exec` subcommands against a container


## Links

- Docker CLI reference:
	- [run](https://docs.docker.com/engine/reference/run/)
	- [exec](https://docs.docker.com/engine/reference/commandline/exec/)
		> The `docker exec` command runs a new command in a running container.
- [What is the different between “run” and “exec”](https://chankongching.wordpress.com/2017/03/17/docker-what-is-the-different-between-run-and-exec/)


## Overview

- Run a container for an image then delete it when stopped.
    ```sh
    $ docker run \
        --rm \
        --name my-app \
        -p 3000:3000 \
        -d \
        --init \
        my-node-img
    ```
- Start an existing container. Omit the `--rm` flag above if you want to have the option to start a stopped container.
    ```sh
    $ docker start my-app
    ```
- Access a running container interactively - useful to run commands and explore the file system.
    ```sh
    $ docker exec -it my-app bash
    ```
    
## Run in a new container

Run a command in a **new** container. The container will then stop when it is finished, unless it continues to run like for a server.

```sh
$ docker run IMAGE
$ docker run IMAGE COMMAND
```

The container will get a different **random** name on each run if you don't specify a name. Warning: you'll end up with a lot of containers unless you use `rm` to delete it each time.

e.g.

This will do nothing and then exit:

```sh
$ docker run python:3.9
```

Use default entry-point (in this case, the Python terminal with `python` command) and use interactive mode:

```sh
$ docker run -it --rm python:3.9
```

Specify Bash shell as entry-point with interactive mode:

```sh
$ docker run -it --rm python:3.9 bash
```

### Run with a name

You can give the container a name. This can be useful if you have long-running container like a server that will be referenced by other containers. Or if you want to install files in the container or test it over multiple days without starting over.

```sh
$ docker run --name CONTAINER_NAME IMAGE
```

e.g.

```sh
$ docker run \
    --name my-app \
    python:3.9
```

### Running with a name and a removing or persisting

If you don't care about state, you can add the `--rm` flag to delete it after it exits (whether you stop it or finishes its task). Then you can use the `run` command with a set name, repeatedly without getting an error that the container exists.

If you want to reuse the same container each time and not get an error that the container name already exists, use `run` without `--rm` and then use `start` as in the section below.

## Run in an existing container

Run a command in an **existing** and **running** container. Give it given a container name or ID.

1. Create the container:
    ```sh
    $ docker run --name CONTAINER_NAME IMAGE
    ```
1. Start it:
    ```sh
    $ docker start CONTAINER_NAME
    ```
1. Execute inside it:
    ```sh
    $ docker exec CONTAINER_NAME
    ```

That is useful if you want to tunnel in and use an interactive session with Bash, Python, etc.

```sh
$ docker exec -it CONTAINER bash
```

If you container exits immediately when it run, you'll struggle to `exec` into it.

So then use `run` against a built image. Set a **new** container from a given image, deleting the container when done. Pass a shell entry point.

```sh
$ docker run --rm -it \
    IMAGE \
    --entrypoint bash
```

In VS Code, under the Docker extension and "Images" you can select an image and "Run interactive" to achieve the same thing. Though, you might start with Node or something else, as you can't specify Bash.


## Interactive

Start an interactive Node console in a target image.

e.g. 

Node image, with `node` as default entrypoint.

```sh
$ docker run -it node
>
```

Node image, with `bash` as custom entrypoint.

```sh
$ docker run -it node bash
#
```


## Open ports

Expose and publish ports.

Format:

- `--publish EXTERNAL:INTERNAL`

With that flag, that you don't need `EXPOSE` in the `Dockerfile`. Also, using `EXPOSE` alone doesn't publish.

Example:

```sh
$ docker run --rm -p 80:8080 node-app
```

The Node app will serve on port `8080` and Docker routes that port `80` on the host.


## Network

From [Network settings](https://docs.docker.com/engine/reference/run/#network-settings).

```
--network="bridge" : Connect a container to a network
                      'bridge': create a network stack on the default Docker bridge
                      'none': no networking
                      'container:<name|id>': reuse another container's network stack
                      'host': use the Docker host network stack
                      '<network-name>|<network-id>': connect to a user-defined network
```

e.g. If `my-network` was created by Docker Compose and you want to add a container to that network.

```sh
$ docker run --network='my-network'
```

## Start shell

Start an interactive Bash shell in the container by setting the entrypoint. This overrides whatever value was set in the `Dockerfile` for `ENTRYPOINT`.

```sh
$ docker run -it my-app bash
```

Or with more verbose `--entrypoint ENTRYPOINT` flag.

Note that flags must come _before_ the app name.

```sh
$ docker run -it --entrypoint bash my-app 
```


## Avoid

- Create - I don't think I've ever had to run `create`.
    ```sh
    $ docker create --init \
        --name my-app \
        -p 3000:3000 \
        my-node-img
    ```
- Attach to a container (not recommended).
    ```sh
    $ docker attach my-app
    ```
