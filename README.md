# docker-ember

Docker and tooling for using ember-cli in a reproducable way.

## Why?

Our ember-cli builds have not been as reproducable as we'd have wanted
them to be.  Tooling can differ across machines because the operating
systems are different, therefore yielding different versions of
nodejs/iojs or different versions of sass bindings.  The sass bindings
are what pushed us over the edge to try out Dockers for sharing our
build environment.

With the advent of user-namespaces in Docker, mounting volumes with
the right privileges has become
transparant. (see http://www.jrslv.com/docker-1-10/#usernamespacesindocker
for some basic info)

The arguments you need to pass to the Docker run command for it to be
useful are too cumbersome, hence we've created scripts to help you
out.

## What?

We have 4 commands, each for a different use-case.  Run them at the
root of your project.

### ed

`ed` is your default friend.  `ed` helps you install npm & bower
dependencies, install new ember dependencies and run any other
non-interactive ember command.

    # Install a dependency
    ed ember install ember-cli-coffeescript
    # Install all current node modules
    ed npm install
    # Install bower components
    ed bower install

### eds

`eds` launches the ember server for you.

    # No nonsense ember server
    eds
    # Proxying to your localhost (note it's been renamed from localhost to host)
    eds --proxy http://host:8080

### edi

`edi` is the interactive version of `ed`.  It can ask you questions
and you can provide interactive answers.

    # Generate a route
    edi ember generate route epic-win
    # Release a new minor version
    edi ember release --minor

### edl
`edl` is your friend when developing addons. It provides a replacement for `npm link` and `npm unlink` that works in docker-ember. 

    # Create a global symlink of your addon
    cd your-ember-addon
    edl
    # Use that addon in another project
    cd your-ember-project
    edl your-ember-addon
    # Remove the global symlink of your addon
    cd your-ember-addon
    edl -u

*Note*: `edl` assumes `edi` is available on your PATH

## How?
Assuming you have docker set up correctly, simply clone this repository and add the bin folder to your path.

```bash
git clone https://github.com/madnificent/docker-ember.git
echo "export PATH=\$PATH:`pwd`/docker-ember/bin" >> ~/.bashrc
source ~/.bashrc
```

### On linux

By default `ed*` commands run as root in the docker container, this means newly created files will be owned as root as well. To avoid this you can use user namespaces to map the container's root user to your own user. This requires some minimal configuration.

Assuming systemd and a username `my-user` the following steps should suffice:

#### 1. Create the correct mapping in `/etc/subuid` and `/etc/subgid`:

```bash
MY_USER_UID=`grep my-user  /etc/passwd | awk -F':' '{ print $3 }'`
MY_USER_GUID=`grep my-user  /etc/passwd | awk -F':' '{ print $4 }'`
echo "ns1:$MY_USER_UID:65536"| sudo tee -a /etc/subuid
echo "ns1:$MY_USER_GUID:65536"| sudo tee -a /etc/subgid
```

#### 2. Adjust ExecStart of docker daemon to include `--userns-remap=ns1`.

For systemd you can use the following command:
```bash
systemctl edit docker.service
```

The config file might look this:
```
ExecStart=
ExecStart=/usr/bin/dockerd --userns-remap=ns1
```

More information on user namespaces is available here:
 * http://docs-stage.docker.com/v1.10/engine/reference/commandline/daemon/#starting-the-daemon-with-user-namespaces-enabled
 * https://docs.oracle.com/cd/E52668_01/E75728/html/ol-docker-userns-remap.html

### On Mac

Docker for Mac creates files under the right username automatically.  Mac does use a login shell when launching the default terminal app, rather than an interactive shell.  These shells don't read the standard ~/.bashrc file, but rather the ~/.bash\_profile file.  Make sure the following is present in your ~/.bash\_profile so ~/.bashrc is always read.

```
if [ -f ~/.bashrc ]; then
   source ~/.bashrc
fi
```
