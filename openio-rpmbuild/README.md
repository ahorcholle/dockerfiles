# What is openio/rpmbuild?

This image provides an easy way to build RPM packages without configuring a dedicated server.
It is build from a [Fedora](https://getfedora.org) 22 distribution, and provide a shell script that wraps the process to create a package, relying on rpmbuild and [Mock](https://fedoraproject.org/wiki/Mock) to build clean packages.

# How to use this image

## Build a package

To build a package, you must specify at least a RPM specfile through the SPECFILE environment variable.  
Default distribution is *epel-7-x86_64*, get the full list in the [Mock source](http://pkgs.fedoraproject.org/cgit/mock.git/).  

```console
# docker run -e SPECFILE=https://raw.githubusercontent.com/open-io/rpm-specfiles/master/python-oiopy/python-oiopy.spec --privileged=true --rm openio/rpmbuild
```
The container should run in [*privileged mode*](http://blog.docker.com/2013/09/docker-can-now-run-within-docker/).

The build script options are available in the [source code](https://github.com/open-io/dockerfiles/blob/master/openio-rpmbuild/build.py).

## Features

### specfile from GIT
If your specfile and your sources are versionned like in Fedora's repository, you can set the SPECFILE parameter to a GIT repository and use `#commit` or `branch` at the end, for example:  

```console
docker run -e DISTRIBUTION=fedora-22-x86_64  -e SPECFILE=http://pkgs.fedoraproject.org/git/bash.git#469c21de51421a30eb99aad8a02148043fcdccce --privileged=true --rm -v ~/docker/local/cache/mock:/var/cache/mock -v ~/docker/local/lib/mock:/var/lib/mock openio/rpmbuild
```

that give the same as:  

```console
docker run -e DISTRIBUTION=fedora-22-x86_64  -e SPECFILE=http://pkgs.fedoraproject.org/git/bash.git?f22 --privileged=true --rm -v ~/docker/local/cache/mock:/var/cache/mock -v ~/docker/local/lib/mock:/var/lib/mock openio/rpmbuild
```

### Use cache
This image relies on Mock that uses a YUM cache to build package faster. As you create a new container from scratch each time you want to build a package, the cache could not be fed.
This image allows you to bind a local directory to keep the cache fetched by Mock in `/var/cache/mock` through your packages build.

Create a local directory:

```console
# docker run -ti -v /var/cache/mock --name rpmbuild-data-container alpine true
```

```console
# docker run -e SPECFILE=https://raw.githubusercontent.com/open-io/rpm-specfiles/master/python-oiopy/python-oiopy.spec --privileged=true --rm --volumes-from rpmbuild-data-container  openio/rpmbuild
```

```console
# docker run -ti --rm --volumes-from rpmbuild-data-container alpine sh
```

### Access the result
You might want to get your packages, the logs or the chroot at the end of the build, you can mount `/var/lib/mock` on your host to access this informations:

Create a local directory:

```console
# mkdir -pv $(pwd)/local/lib/mock
```

```console
# docker run -e SPECFILE=https://raw.githubusercontent.com/open-io/rpm-specfiles/master/python-oiopy/python-oiopy.spec --privileged=true -v local/lib/mock:/var/lib/mock --rm openio/rpmbuild
```

# License

The included script is provided under [Apache License v2](http://www.apache.org/licenses/LICENSE-2.0).