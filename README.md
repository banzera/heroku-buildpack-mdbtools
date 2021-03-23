heroku-buildpack-mdbtools
=================================

Use [mdbtools](https://github.com/mdbtools/mdbtools) inside a Heroku `heroku-20` environment. 

The tar file in the [/build folder](./build) currently contains: 

Version: mdbtools 0.9.2 

You will need to build a new binary if you want to use a newer or different version. To build a new binary see [How to Build a New Binary](#how-to-build-a-new-binary)

## Usage

**NOTE:** _To ensure the newer version of mdbtools is found in the $PATH and installed first make sure this buildpack is added to the top of the buildpack list or at "index 1"._


From your projects "Settings" tab add this buildpack to your app in the 1st position:

```
https://github.com/banzera/heroku-buildpack-mdbtools
```

**OR**

From the command line:

```
heroku buildpacks:add https://github.com/banzera/heroku-buildpack-mdbtools --index 1 --app HEROKU_APP_NAME
```

## How to Build a New Binary

The binary in this repo was built in a heroku:20 docker image running in a local dev environment. The tar file was then copied into the `/build` directory in this repo and is used by the [compile script](./bin/compile).

Prerequisites

- Docker installed and running in local dev environment. [Get Docker](https://docs.docker.com/get-docker/)

Steps:

1. Spin up a docker container with the heroku:16 stack. This will build and behave exactly the same way a heroku:16 dyno except you will have write access. (make sure docker is running on your machine). From command line:
 
     ```bash
     $ docker run --rm -it heroku/heroku:20-build
     ```
 
_This will take you to an interactive bash shell as a root user inside the container. The `--rm` flag removes the docker process on exiting.  The `-ti` flag creates the interactive bash shell._
 
 
2. Get the libraries and dependencies you need(some of these already exist on the system):

     ```bash
     $ apt-get update && apt-get install build-essential autoconf libtool git gettext
     ```

3. Get, Configure and Install Newest Imagemagick:

    ```bash
    $ wget https://github.com/mdbtools/mdbtools/releases/download/v0.9.2/mdbtools-0.9.2.tar.gz
    $ tar xf mdbtools-0.9.2.tar.gz
    $ cd mdbtools/
    $ autoreconf -i -f
    $ ./configure --disable-silent-rules --${{ matrix.glib }} --with-unixodbc=/usr
    $ make
    ```

_Take a break this will take a few min to install._

10. Wrap it up with a bow(compress the binary):

    ```bash
    $ cd /
    $ tar czf /mdbtools.tar.gz mdbtools/
    ```


11. Copy the compressed file/tarball from the docker container into the repo(_you need to have cloned this repo locally_):
 
    ```bash
    # List current running docker processes to find out the NAME of your container
    $ docker ps
    # copy the binary from the container to the build directory in the repo on your local machine
    $ docker cp <NAME_of_docker_container>:/mdbtools.tar.gz <path_to_build_folder_in_git_repo>
    ```
     

**DO NOT EXIT YOUR CONTAINER or ALL will be lost, open a new tab in your terminal**
 
_You may need to delete the old tarball from the bin folder first or to be safe copy the file from the container to your local machine before adding to the repo so you have a copy of the old binary tarball._


12. Commit and Push to repo


### Clear cache(_Not Sure if this is necessary)
Since the installation is cached you might want to clean it out due to config changes.

1. `heroku plugins:install heroku-repo`
2. `heroku repo:purge_cache -app HEROKU_APP_NAME`

### Credits
https://medium.com/@eplt/5-minutes-to-install-imagemagick-with-heic-support-on-ubuntu-18-04-digitalocean-fe2d09dcef1
https://github.com/brandoncc/heroku-buildpack-vips
https://github.com/steeple-dev/heroku-buildpack-imagemagick
https://github.com/retailzipline/heroku-buildpack-imagemagick-heif
