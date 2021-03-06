Automotive Grade Linux HTML5 Homescreen
=======================================

An AGL Homescreen application written in HTML5 / angular2 / TypeScript

## Installation

Install HTML5 development toolchain on your host

Install NodeJs *[not used on target]*

```bash
 zypper install nodejs
 yum install nodejs
 sudo apt install nodejs
```

Install building tools

```bash
 sudo npm install --global @angular/cli # Angular Command Line Interface
```

## Build project

### Clone the repository

`git clone -b iotbzh https://github.com/iotbzh/agl-homescreen.git`

**Pay attention to use the *iotbzh* branch.**

and navigate to `agl-homescreen` directory:

```bash
cd agl-homescreen
```

Install dependencies by running the following commands

```bash
npm install
```

`node_modules` directory will be created during the install.

### Building the project in development mode

### Connect to the board

```bash
ls -altr /dev
screen /dev/tty.usbserial-AK04WW00 115200
root
ip a
```

#### Development mode on host with (fallback) dev-server
### SSH

```bash
ssh root@IP
./launcher.sh

/usr/bin/afb-daemon --port=8000 --rootdir=/home/root/agl-homescreen --token=123456789 --roothttp=. --alias=/icons:/var/local/lib/afm/icons &
/usr/bin/google-chrome --mus --no-sandbox --test-type --kiosk http://localhost:8000
```

  - Setup `sync` task in `package.json` file:
```bash
rsync -avrzh --rsh=\"ssh -p 4444 -l root\" ./dist/. root@localhost:agl-homescreen
```

  - Setup `src/environments/environment.ts` file:
```bash
    ....
	service: {
        ip: null,           // dynamically set when set to null (see main.ts)
        port: 5000,         // dynamically set when set to null (see main.ts)
        api_url: "/api",
    },
    ....
```

  - Start compile code and start dev-server
```bash
    npm run start  # to start up dev-server (AKA fake server)
```

  - Open [http://localhost:8000](http://localhost:8000) in a browser.

*Note:* Use [http://localhost:8000/#/event-emitter](http://localhost:8000/#/event-emitter) url for emitting login/logout events

#### Development mode deployed on a target

You can use a real target (raspi3, porter, ...) or a qemu.

##### For example, use the following instructions for qemux86-64 running in VirtualBox:

  - Download qemux86-64 vmdk image : [agl-demo-platform-qemux86-64.vmdk](https://download.automotivelinux.org/AGL/snapshots/master/latest/qemux86-64/deploy/images/qemux86-64/)

  - Download (or build) af-main-binding library : [af-main-binding-1.0-r0.core2_64.rpm](qemux86-64/build/tmp/deploy/rpm/./core2_64/af-main-binding-1.0-r0.core2_64.rpm)

  - Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads), for example on Ubuntu:
```bash
    sudo sh -c 'echo "deb http://download.virtualbox.org/virtualbox/debian xenial contrib" >> /etc/apt/sources.list.d/virtualbox.list'
    wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
    sudo apt update
    sudo apt install virtualbox-5.1
```

  - Setup a VirtualBox machine with the following settings:
    - `Type`: Linux
    - `Version`: Linux 2.6 / 3.x / 4.x (64-bit)

  - Use `existing virtual hard disk file` and select the previous downloaded vmdk image.

  - Network settings: `Adapter 1: NAT`, in Advanced menu, click on `Port Forwarding` and define the following rules:

    | Name | Protocol | Host IP | Host Port | Guest IP | Guest Port |
    |------|----------|---------|-----------|----------|------------|
    | ssh  |    TCP   |         |   4444    |          |      22    |
    | svr  |    TCP   |         |   8000    |          |    8000    |


  - Setup `sync` task in `package.json` file:
```bash
rsync -avrzh --rsh=\"ssh -p 4444 -l root\" ./dist/. root@localhost:agl-homescreen
```

  - Compile and and deploy code on target using:
```bash
  npm run deploy
```

  - Start app on target:
```bash
    ssh -p 4444 root@localhost

    # Manually install afm-main (only if needed and only the first time)
    rpm -ihv af-main-binding-1.0-r0.core2_64.rpm

    # Start app on target
    /usr/bin/afb-daemon --port=8000 --rootdir=/home/root/agl-homescreen --token=123456789 --roothttp=. --alias=/icons:/var/local/lib/afm/icons
```

> NOTES: for old AGL version (previous DD version), command line should be:
>```bash
> /usr/bin/afb-daemon --port=8000 --rootdir=/home/root/agl-homescreen --mode=remote --token=123456789 --roothttp=. --alias=/icons:/var/lib/afm/icons --sessiondir=/tmp/.afb-daemon
>```

    Following options may be added to `afb-daemon` command to help debugging:
    `-vvv` and/or `--tracereq=all`

  - Open [http://localhost:8000](http://localhost:8000) in a browser.

##### Run `qemu` (without VirtualBox) :

  - Build AGL qemux86-64 image and run it (please refer to [AGL getting started](http://docs.automotivelinux.org/docs/getting_started/en/dev/reference/machines/qemu.html) for more info) :
```bash
    source meta-agl/scripts/aglsetup.sh -m qemux86-64 agl-demo agl-netboot agl-appfw-smack
    bitbake agl-demo-platform
    runqemu qemux86-64
```
  - Setup `sync` task in `package.json` file:
```bash
    rsync -avrzh --rsh=\"ssh -l root\" ./dist/. root@192.168.7.2:agl-homescreen
```

  - Compile and and deploy code on target using:
```bash
    npm run deploy
```

  - Start app on target:
```bash
    ssh root@192.168.7.2

    # Manually install afm-main (only if needed and only the first time)
    rpm -ihv af-main-binding-1.0-r0.core2_64.rpm

    # Start app on target
    /usr/bin/afb-daemon --port=8000 --rootdir=/home/root/agl-homescreen  --sessiondir=/tmp/.afb-daemon --mode=remote --token=123456789 --roothttp=. --alias=/icons:/var/lib/afm/icons
```

  - Open [http://192.168.7.2:8000](http://192.168.7.2:8000) in a browser.


#### Production mode

*Should be reworked...*

- Create a build folder for distribution (please follow steps from development mode for that)
- Serve the build folder with an apache or any other web server
    - copy content of build folder in /var/www/html/
    - add line FallbackResource /html/index.html in apache2.conf


### TODO

- implement getDetails to populate detailsApp
- add a terminal (ssh like) to interact with target shell
- Fix HVAC background image: it is not the right one

### FAQ

#### Error: can't start the binding path set /usr/lib/afb

if you encounter this kind of error when starting `afm-daemon` on target:
```bash
ERROR: binding [/usr/lib/afb/afm-main-binding.so] register function failed. continuing...
ERROR: can't start the binding path set /usr/lib/afb
```

You must define `DBUS_SESSION_BUS_ADDRESS` variable as follow:

```bash
export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/0/bus
```

## Useful links

- Building AGL image: [http://docs.iot.bzh/docs/getting_started/en/dev/](http://docs.iot.bzh/docs/getting_started/en/dev/)
- AGL application framework documentation: [http://docs.iot.bzh/docs/apis_services/en/dev/reference/af-main/overview.html](http://docs.iot.bzh/docs/apis_services/en/dev/reference/af-main/overview.html)
