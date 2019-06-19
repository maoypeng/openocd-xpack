# How to build the xPack OpenOCD?

## Introduction

This project also includes the scripts and additional files required to 
build and publish the
[xPack OpenOCD](https://github.com/xpack-dev-tools/openocd-xpack).

The build scripts use the
[xPack Build Box (XBB)](https://github.com/xpack/xpack-build-box), 
a set of elaborate build environments based on GCC 7.4 (Docker containers
for GNU/Linux and Windows or a custom folder for MacOS).

## Repository URLs

- https://github.com/xpack-dev-tools/openocd.git - the URL of the 
  [xPack OpenOCD fork](https://github.com/xpack-dev-tools/openocd)
- git://git.code.sf.net/p/openocd/code - the URL of the 
  [upstream OpenOCD](http://openocd.org).

The build scripts use the first; to merge
changes from upstream it is necessary to add a remote named
`upstream`, and merge the upstream master into the local master.

## Branches

- xpack - the updated content, used during the builds
- master - the original content; it follows the upstream master.

## Download the build scripts

The build scripts are available in the `scripts` folder of the 
[xpack-dev-tools/openocd-xpack](https://github.com/xpack-dev-tools/openocd-xpack) 
Git repo.

To download them, the following shortcut is available: 

```console
$ curl -L https://github.com/xpack-dev-tools/openocd-xpack/raw/master/scripts/git-clone.sh | bash
```

The small script issues the following two commands:

```console
$ rm -rf ~/Downloads/openocd-xpack.git
$ git clone --recurse-submodules https://github.com/xpack-dev-tools/openocd-xpack.git \
~/Downloads/openocd-xpack.git
```

> Note: the repository uses submodules; for a successful build it is 
> mandatory to recurse the submodules.

To use the `develop` branch of the build scripts, use:

```console
$ rm -rf ~/Downloads/openocd-xpack.git
$ git clone --recurse-submodules -b develop https://github.com/xpack-dev-tools/openocd-xpack.git \
~/Downloads/openocd-xpack.git
```

## The `Work` folder

The scripts create a temporary build `Work/openocd-${version}` folder in 
the user home. Although not recommended, if for any reasons you need to 
change the location of the `Work` folder, 
you can redefine `WORK_FOLDER_PATH` variable before invoking the script.

## Changes

Compared to the original OpenOCD distribution, there should be no 
functional changes. 

The actual changes for each version are documented in the 
`scripts/README-<version>.md` files.

## How to build local/native binaries

TBD

## How to build distributions

### Prerequisites

The prerequisites are common to all binary builds. Please follow the 
instructions in the separate 
[Prerequisites for building binaries](https://gnu-mcu-eclipse.github.io/developer/build-binaries-prerequisites-xbb/) 
page and return when ready.

### Preload the Docker images ()

Docker does not require to explicitly download new images; it does this 
automatically at first use.

However, since the images used for this build are relatively large, it 
is recommended to load them explicitly before starting the build:

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh preload-images
```

The result should look similar to:

```console
$ docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
ilegeul/centos32 6-xbb-v2.2 0a2b918ea04e 4 weeks ago 3.03GB
ilegeul/centos 6-xbb-v2.2 4b0354b70743 4 weeks ago 3.12GB
hello-world latest f2a91732366c 2 months ago 1.85kB
```

### Update git repos

To keep the development repository in sync with the original OpenOCD 
repository and the RISC-V repository:

- checkout `master`
- pull from `openocd/master`
- checkout `xpack-dev-tools-dev`
- merge `master`
- add a tag like `v0.10.0-8` after each public release (mind the 
inner version `-8`)

### Prepare release

To prepare a new release, first determine the OpenOCD version 
(like `0.10.0`) and update the `scripts/VERSION` file. The format is 
`0.10.0-8`. The fourth digit is the xPack release number 
of this version.

Add a new set of definitions in the `scripts/container-build.sh`, with 
the versions of various components.

### Update README.md

If necessary, update the main `README.md` with informations related to the
build. Information related to the content should be updated in 
the `README-<version>.md`.

### Create README-&lt;version&gt;.md

Create a copy of the previous one and update.

### Update CHANGELOG.md

Check `openocd-xpack.git/CHANGELOG.md` and add the new release.

### Build

Although it is perfectly possible to build all binaries in a single step 
on a macOS system, due to Docker specifics, it is faster to build the 
GNU/Linux and Windows binaries on a GNU/Linux system and the macOS binary 
separately on a macOS system.

#### Build the GNU/Linux and Windows binaries

The current platform for GNU/Linux and Windows production builds is an 
Ubuntu Server 18 LTS running on an Intel NUC8i7BEH mini PC with 32 GB of RAM 
and 512 GB of fast M.2 SSD.

```console
$ ssh ilg-xbb-linux
```

Before starting a multi-platform build, check if Docker is started:

```console
$ docker info
```

Before the first time build, it is recommended to preload the
docker images.

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh preload-images
```

The result should look similar to:

```console
$ docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ilegeul/centos32    6-xbb-v2.2          956eb2963946        5 weeks ago         3.03GB
ilegeul/centos      6-xbb-v2.2          6b1234f2ac44        5 weeks ago         3.12GB
hello-world         latest              fce289e99eb9        5 months ago        1.84kB
```

To build both the 32/64-bit Windows and GNU/Linux versions, use `--all`; 
to build selectively, use any combination of
`--linux64`, `--win64`, `--linux32` or `--win32` 

Since the build takes a while, use `screen` to isolate the build session.

```console
$ screen -S openocd

$ sudo rm -rf ~/Work/openocd-*
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh --all
```

Some time later, the output of the build script is a set of 4 files 
and their SHA signatures, created in the `deploy` folder:

```console
$ ls -l deploy
total 10940
-rw-r--r-- 1 ilg ilg 2655896 May 12 22:39 xpack-openocd-0.10.0-12-centos32.tgz
-rw-r--r-- 1 ilg ilg 126 May 12 22:39 xpack-openocd-0.10.0-12-centos32.tgz.sha
-rw-r--r-- 1 ilg ilg 2590467 May 12 22:27 xpack-openocd-0.10.0-12-centos64.tgz
-rw-r--r-- 1 ilg ilg 126 May 12 22:27 xpack-openocd-0.10.0-12-centos64.tgz.sha
-rw-r--r-- 1 ilg ilg 2910732 May 12 22:45 xpack-openocd-0.10.0-win32.zip
-rw-r--r-- 1 ilg ilg 123 May 12 22:45 xpack-openocd-0.10.0-win32.zip.sha
-rw-r--r-- 1 ilg ilg 2957245 May 12 22:34 xpack-openocd-0.10.0-win64.zip
-rw-r--r-- 1 ilg ilg 123 May 12 22:34 xpack-openocd-0.10.0-win64.zip.sha
```

To copy the files from the build machine to the current development 
machine, either use NFS to mount the entire folder, or open the `deploy` 
folder in a terminal and use `scp`:

```console
$ cd ~/Work/openocd-*/deploy
$ scp * ilg@ilg-mbp.local:Downloads/gme-binaries/oocd
```

#### Build the macOS binary

The current platform for macOS production builds is a macOS 10.10.5 
VirtualBox image running on a macMini with 16 GB of RAM and a 
fast SSD.

To build the latest macOS version, with the same timestamp as the 
previous build:

```console
$ rm -rf ~/Work/openocd-*
$ caffeinate bash ~/Downloads/openocd-xpack.git/scripts/build.sh --osx
```

For consistency reasons, the date should be the same as the GNU/Linux 
and Windows builds.

Some time later, the output of the build script is a compressed 
archive and its SHA signature, created in the `deploy` folder:

```console
$ ls -l deploy
total 4944
-rw-r--r-- 1 ilg staff 2524910 May 12 23:19 xpack-openocd-0.10.0-macos.tgz
-rw-r--r-- 1 ilg staff 123 May 12 23:19 xpack-openocd-0.10.0-macos.tgz.sha
```

To copy the files from the build machine to the current development 
machine, open the `deploy` folder in a terminal and use `scp`:

```console
$ cd ~/Work/openocd-*/deploy
$ scp * ilg@ilg-mbp.local:Downloads/gme-binaries/oocd
```

### Subsequent runs

#### Separate platform specific builds

Instead of `--all`, you can use any combination of:

```
--win32 --win64 --linux32 --linux64
```

Please note that, due to the specifics of the GCC build process, the 
Windows build requires the corresponding GNU/Linux build, so `--win32` 
alone is equivalent to `--linux32 --win32` and `--win64` alone is 
equivalent to `--linux64 --win64`.

#### clean

To remove most build temporary files, use:

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh --all clean
```

To also remove the library build temporary files, use:

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh --all cleanlibs
```

To remove all temporary files, use:

```console
$ bash ~/Downloads/openocd-xpack.git/scripts/build.sh --all cleanall
```

Instead of `--all`, any combination of `--win32 --win64 --linux32 --linux64`
will remove the more specific folders.

For production builds it is recommended to completely remove the build folder.

#### --develop

For performance reasons, the actual build folders are internal to each 
Docker run, and are not persistent. This gives the best speed, but has 
the disadvantage that interrupted builds cannot be resumed.

For development builds, it is possible to define the build folders in 
the host file system, and resume an interrupted build.

#### --debug

For development builds, it is also possible to create everything with 
`-g -O0` and be able to run debug sessions.

#### Interrupted builds

The Docker scripts run with root privileges. This is generally not a 
problem, since at the end of the script the output files are reassigned 
to the actual user.

However, for an interrupted build, this step is skipped, and files in 
the install folder will remain owned by root. Thus, before removing 
the build folder, it might be necessary to run a recursive `chown`.

## Install

The procedure to install GNU MCU Eclipse OpenOCD is platform specific, 
but relatively straight forward (a .zip archive on Windows, a compressed 
tar archive on macOS and GNU/Linux).

A portable method is to use [`xpm`](https://www.npmjs.com/package/xpm):

```console
$ xpm install --global @xpack-dev-tools/openocd
```

More details are available on the 
[How to install the OpenOCD binaries?](https://xpack-dev-tools.github.io/openocd/install/) page.

After install, the package should create a structure like this (only the 
first two depth levels are shown):

```console
$ tree -L 2 /Users/ilg/Library/xPacks/\@xpack-dev-tools/openocd/0.10.0-8.1/.content/
/Users/ilg/Library/xPacks/\@xpack-dev-tools/openocd/0.10.0-8.1/.content/
├── OpenULINK
│   └── ulink_firmware.hex
├── README.md
├── bin
│   └── openocd
├── contrib
│   ├── 60-openocd.rules
│   └── libdcc
├── xpack-dev-tools
│   ├── CHANGELOG.txt
│   ├── licenses
│   ├── patches
│   └── scripts
├── scripts
│   ├── bitsbytes.tcl
│   ├── board
│   ├── chip
│   ├── cpld
│   ├── cpu
│   ├── fpga
│   ├── interface
│   ├── mem_helper.tcl
│   ├── memory.tcl
│   ├── mmr_helpers.tcl
│   ├── target
│   ├── test
│   └── tools
└── share
└── doc
```

No other files are installed in any system folders or other locations.

## Uninstall

The binaries are distributed as portable archives; thus they do not need 
to run a setup and do not require an uninstall.

## Test

A simple test is performed by the script at the end, by launching the 
executable to check if all shared/dynamic libraries are correctly used.

For a true test you need to first install the package and then run the 
program from the final location. For example on macOS the output should 
look like:

```console
$ /Users/ilg/Library/xPacks/\@xpack-dev-tools/openocd/0.10.0-8.1/.content/bin/openocd --version
xPack 64-bit Open On-Chip Debugger 0.10.0+dev-00487-gaf359c18 (2018-05-12-23:16)
```

## More build details

The build process is split into several scripts. The build starts on 
the host, with `build.sh`, which runs `container-build.sh` several 
times, once for each target, in one of the two docker containers. 
Both scripts include several other helper scripts. The entire process 
is quite complex, and an attempt to explain its functionality in a few 
words would not be realistic. Thus, the authoritative source of details 
remains the source code.

## License

The original content is released under the 
[MIT License](https://opensource.org/licenses/MIT), with all rights reserved to
[Liviu Ionescu](https://github.com/ilg-ul).




## Download analytics

* GitHub [xpack-dev-tools/openocd.git](https://github.com/xpack-dev-tools/openocd/)
* latest release
[![Github All Releases](https://img.shields.io/github/downloads/xpack-dev-tools/openocd/latest/total.svg)](https://github.com/xpack-dev-tools/openocd/releases/)
* all releases [![Github All Releases](https://img.shields.io/github/downloads/xpack-dev-tools/openocd/total.svg)](https://github.com/xpack-dev-tools/openocd/releases/)
* xPack [@xpack-dev-tools/openocd](https://github.com/xpack-dev-tools/openocd-xpack/)
* latest release, per month 
[![npm (scoped)](https://img.shields.io/npm/v/@xpack-dev-tools/openocd.svg)](https://www.npmjs.com/package/@xpack-dev-tools/openocd/)
[![npm](https://img.shields.io/npm/dm/@xpack-dev-tools/openocd.svg)](https://www.npmjs.com/package/@xpack-dev-tools/openocd/)
* all releases [![npm](https://img.shields.io/npm/dt/@xpack-dev-tools/openocd.svg)](https://www.npmjs.com/package/@xpack-dev-tools/openocd/)
* [individual file counters](https://www.somsubhra.com/github-release-stats/?username=xpack-dev-tools&repository=openocd) (grouped per release)

Credit to [Shields IO](https://shields.io) and [Somsubhra/github-release-stats](https://github.com/Somsubhra/github-release-stats).


