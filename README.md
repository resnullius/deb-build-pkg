deb-build-pkg
=============

This small script builds `.deb` packages using docker. Is a quick and easy way to
create `.deb`s. It only requires the regular packaging structure for a `.deb`
with a `/debian` directory at the root of the source to be packaged. Here's a
[guide from Debian on how to create `.deb`
packages](https://www.debian.org/doc/manuals/maint-guide/).

Most methods for building packages requires the developer to be on a Debian
system in the unstable release, yet, with this script you can be in any release
or any other operative system as long as you have docker and bash (which is the
language the script is written in).

Using it is quite simple, for building a package for debian unstable you would
only need to run this command from within the root directory of the source:

    deb-build-pkg build

Then if you go to `$PWD/../pkgs/` you'll find the output of the build which
consists on the `.deb` packages, a `.dsc`, a `.changes`, the `.orig.tar.gz` and
the original file where that `.orig.tar.gz` comes from.

If you're building for uploading into Debian, Ubuntu or just for installing in
your system this is more than enough.


<!-- vim-markdown-toc GFM -->

* [Packing for quick-release](#packing-for-quick-release)
* [Adding extra repositories for build-time](#adding-extra-repositories-for-build-time)
* [Why would I want to use docker for building packages?](#why-would-i-want-to-use-docker-for-building-packages)
* [Building the non-standard way](#building-the-non-standard-way)
* [How does it actually works? (In the inside)](#how-does-it-actually-works-in-the-inside)
* [License](#license)

<!-- vim-markdown-toc -->

Packing for quick-release
-------------------------

If you're a particular developer of some software, it's very likely you won't be
uploading the packages to Debian or Ubuntu but distributing them in a compressed
format via something like a GitHub release. For that there are two commands
built into `deb-build-pkg`: `pack` and `bundle`.

If you're like me, it's very likely you'll build something several times, test
the final result is what you expect and then release it, for that there's
`deb-build-pkg pack` which takes the files from `$PWD/../pkgs` and throws them
into a `.tar.xz` with the naming scheme of `$distribution-$image_ver-$arch`, so
something like `debian-unstable-x86_64.tar.xz` and leave it in the
`$PWD/../release` directory.

If you're the kind of people that builds and forgets, then probably
`deb-build-pkg bundle` is the one you're looking for. It will build **and** pack
in the same command.

Adding extra repositories for build-time
----------------------------------------

If you need extra repositories besides the official ones from the distribution
you're building the packages for, then you can use `-x` to add this extra
repositories on-the-fly. This would look like:

    deb-build-pkg build \
      -x "https://extra.repo.org/repo unstable main"

You can see this is mostly the same string that you would put in your
`/etc/apt/sources.list` but is missing the `deb ` at the beggining, and indeed
that's the case.

You can add multiple extra repositories by calling `-x` several times, like:

    deb-build-pkg build \
      -x "https://extra.repo.org/repo unstable main" \
      -x "https://extra2.repo.org/repo unstable main"

It's very likely you would like to add an extra GPG key in order to validate the
packages from the extra repositories, in that case you have the `-g` argument,
so adding the `0xB9AC83EE` GPG key, you would need to call `deb-build-pkg` like:

    deb-build-pkg build \
      -x "https://extra.repo.org/repo unstable main" \
      -x "https://extra2.repo.org/repo unstable main" \
      -g 0xB9AC83EE

The use case for this is when you're building your own set of packages that
depend each other, so in order to build package B you need A, which you upload
to one of your own repositories and then make it available to B so it satisfies
that dependency.

Why would I want to use docker for building packages?
-----------------------------------------------------

With `deb-build-pkg` you always build things in a clean, blank system. The
docker images used by this script by default are the bare images provided by
debian and ubuntu itself plus these packages: `build-essential`, `equivs`,
`devscripts`, `pristine-tar` and `git`. There won't be "it works for me, why
doesn't work there?", there won't be messy development systems with different
versions of libraries half-baked installed in your machine.

The docker image itself is stateless, so any installation or modification made
during the execution of the building process won't be saved and the next run
will be on a clean slate.

Using docker also means you don't need to have a host in the same operative
system that the one for your package, so you can build for debian stable while
being on debian unstable, or even another operative system altogether like
macOS. You only need docker and bash.

Building the non-standard way
-----------------------------

In Debian is expected to put things into a branch called `debian/master` and the
source for the code comes in the branch called `master`, but what I like is to
keep `master` for the upstream source code and have several versions in
different branches, so `debian/unstable` is for unstable, `ubuntu/trusty` is for
trusty in ubuntu, `debian/stable` is for debian stable... You see where I'm
going with this.

That way I can keep debian packages for multiple distros in the same git
repository.

That's what the `-n` argument works for. When is this useful? The reason for
building this tool is that in my `$day_job` at the moment they needed to have a
set of applications up-to-date in different versions of debian and ubuntu, so
with the help of this tool we would build the packages for each distro and
release, then upload to a private repository and pass that to the clients.

In most cases you won't need the non-standard way and you're good to go with
`-n` but in case you need it, it exists. This is an example of how a [git
repository for the non-standard way would look
like](https://github.com/resnullius/capstone.deb).

How does it actually works? (In the inside)
-------------------------------------------

The person reading this section is probably the curious that read the script and
can see there's nothing in it calling `dpkg-buildpackage` or anything, and
you're right. All the magic happens inside the docker images that come from the
[`resnullius/docker-deb-devel`](https://github.com/resnullius/docker-deb-devel)
git repository. If you want to get a more in-depth explanation go there, but the
TL;DR; version of that is: there's a base script called `entrypoint.sh` where
all the magic happens: adding gpg keys, adding extra repositories, downloading
upstream source code, installing dependencies, building the packages and copying
the packages to the output directory linked as a volume from the image to your
host.

This script is just a frontend for that, so you don't need to call the docker
image directly with `docker run ...` and all the variables and stuff. Actually,
for a regular debian developer or maintainer it has all the right defaults so
it is even less things to write/call.

License
-------

© 2016-2019, Jose-Luis Rivas `<me@ghostbar.co>`. Released under the MIT license
terms.
