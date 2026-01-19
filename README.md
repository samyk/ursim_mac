# Run ursim (Universal Robots Simulator) on Apple Silicon/ARM Mac

## Quick Setup

```sh
# install rosetta if needed
ls /Library/Apple/usr/libexec/oah/libRosettaRuntime || softwareupdate --install-rosetta --agree-to-license

# install docker and brew if needed
docker -v || (brew -v || /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" && brew install docker)

PROGRAM_DIR="${HOME}/Documents/programs"
mkdir -p "$PROGRAM_DIR"

# robot model
# CB3 models: UR3 UR5 UR10
# E-series models: UR3 UR5 UR7 UR8LONG UR10 UR12 UR15 UR16 UR18 UR20 UR30
MODEL=UR5

# cb3 or e-series?
#CONTAINER=universalrobots/ursim_cb3
CONTAINER=universalrobots/ursim_e-series

echo 'once running, open VNC client to 127.0.0.1:5901 or browse to http://127.0.0.1:6080/vnc.html?host=127.0.0.1&port=6080'
docker run --rm -it -p 5901:5900 -p 6080:6080 -e ROBOT_MODEL=$MODEL -v "$PROGRAM_DIR:/ursim/programs" --platform=linux/amd64 $CONTAINER

# now open VNC client to 127.0.0.1:5901 or browse to http://127.0.0.1:6080/vnc.html?host=127.0.0.1&port=6080
# you may need to ignore the IP addresses the docker suggests...
```

## Setup Considerations

As of Jan 2025, [Universal Robots E-Series Offline Simulator](https://www.universal-robots.com/download/software-ur20ur30/simulator-linux/offline-simulator-e-series-and-ur20ur30-ur-sim-for-linux-5200/) is a software written in Java with some x86 32 bits binary (URControl). If it was solely in Java, it could run on ARM processors just by having the right Java JVM on ARM, but the x86 32 bits binary requires emulation of the x86 processor for the ARM computers. [VirtualBox](https://www.virtualbox.org), [VMWare](https://www.vmware.com) and [Parallels](https://www.parallels.com) did not have any x86 emulation at that time. We tried many possibilities such as using the [UTM](https://getutm.app) VM software on Mac with QEMU x86 emulation, but it was way way too slow. We tried also UTM with Apple Virtualization and Rosetta emulation, but it almost worked (URSim started, but got stuck).  We had more success with Docker and it was pretty easy once we found the solution to use the Docker image provided by Universal Robotics.

Therefore these instructions are based on running the Docker image provided by Universal Robotics, tell docker it is a amd64 image, and let Apple Rosetta do the x86 emulation. The performance, while far from a native ARM application, was sufficient. Hence the instructions below.

## Steps

- Install Rosetta on your Mac, which does Intel x86 emulation on ARM:
  - In Terminal: `softwareupdate --install-rosetta --agree-to-license`
- Install Docker
  - either [Docker Desktop](https://www.docker.com/products/docker-desktop/) for Mac - Apple Silicon. Install and launch it. (No need to have a user logged into the Docker network)
  - or in macOS Terminal> `brew install docker`
- In the Docker Terminal window (button on the bottom right of the main window) or in the macOS Terminal if you use brew, type (in one line):
  - `docker run --rm -it -p 5901:5900 -p 6080:6080 -v "${HOME}/Documents/programs:/ursim/programs" --platform=linux/amd64 universalrobots/ursim_e-series`
  - the x86 to ARM emulation is done by the underlying Rosetta but we tell Docker that the image is an amd64 with the `--platform` argument. One can use the exact same instructions here for Mac Intel, Windows or Linux by just not adding the `--platform` argument.
  - the -p arguments are used to expose ports from docker to the host computer. 5900 is for VNC but may already be used by launchd so we map it to 5901 locally. 6080 is for http.
  - the -v argument is to map a directory within the docker with a directory on the mac.
- in a [VNC client](https://www.realvnc.com/en/connect/download/viewer/), connect to `127.0.0.1:5901` or browse to [http://127.0.0.1:6080/vnc.html?host=127.0.0.1&port=6080](http://127.0.0.1:6080/vnc.html?host=127.0.0.1&port=6080)
- URSim should show on your VNC client screen or browser

## Usage Considerations

### Directory Sharing

Docker are lightweight ephemeral instances. When a docker instance is terminated (by the user typing Ctrl-C or accidently), all files are gone and cannot be retrieved. When launching a new docker instance, it starts a fresh new one. Therefore, all user files, profiles, state of the software are lost when an instance is terminated. To mitigate that, one can add directory sharing between the docker instance and the host (your Mac) or use Docker volumes. The former is what is proposed in these instructions (-v "${HOME}/Documents/programs:/ursim/programs") which maps the Mac directory Documents/programs as /ursim/programs into the docker instance, a directory known by the URSim software. Therefore, before terminating your docker instance, make sure all the files you need next time are saved in the shared directory (within docker, per the instructions above, this is the /ursim/programs directory). One can also put files on the Mac directory (Documents/programs) such as URSim programs and they will be seen by the URSim simulator in the docker instance.

### VNC vs HTTP

You can use a browser instead of VNC by browsing to [http://127.0.0.1:6080/vnc.html?host=127.0.0.1&port=6080](http://127.0.0.1:6080/vnc.html?host=127.0.0.1&port=6080)

### macOS Version

This procedure seemed to work on macOS Sonoma 14.2, but the URSim software hang. On that laptop running macOS Sonoma, we upgraded it to macOS Sequoia 15.2 and without reinstalling anything, it just worked.

## More Information

Additional information for more tailored docker setup is available at the [URSIM docker image information](https://hub.docker.com/r/universalrobots/ursim_e-series)

## Virtual Machine Version

We tried to make a VM version, without luck. Only [UTM](https://getutm.app) was able to do emulation with Rosetta, following these [instructions](https://docs.getutm.app/advanced/rosetta/). We also tried the [Debian preconfigured VM for Rosetta](https://mac.getutm.app/gallery/debian-12-rosetta) and installed URSim and adjusting many parameters in the script, again without luck. It almost worked, as Polyscope loaded but freezed, most likely because URControl crashed. We think it could be possible to make it run under a VM, but software being binary, it is like a black box and difficult to troubleshoot. It would be easier if Universal Robots update its software for modern environments. 

## Docker Compose

terryc on discord Universal Robots channel shared its docker compose file, which is copied [here](docker-compose.yml). However, we haven't tested it.

## Comments

Pull requests and issues welcome.

Originally by [Marc Blanchet](https://github.com/marcblanchet/ursim_on_mac_apple_silicon). Thanks to Julie Blanchet and Guillaume Blanchet for hints and testing of these instructions. Updates by [samy kamkar](https://github.com/samyk/ursim_mac).
