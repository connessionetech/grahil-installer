# Grahil Installer

## Introduction

`Grahil installer` is a collection of scripts to facilitate smooth and smart installation, update and uninstall of the `grahil` software on popular desktop/server linux systems as well as ARM systems such as Raspberry PI/Orange PI. The script is optimized to detect your system type, and install all necessary dependencies thta are needed to run `Grahil`. The installer will propmpt for superuser permissions when it is started. This permission is used to execute `sudo` level commands.

`Grahil installer` makes use of root's crontab to register itself as an autoupdater. The shell script is then autorun by the system once a day (at designated hour) to look for and autoupdate the exsiting `grahil` installation. It is important to note that Updating can be done manually as well as automatically. We will be discussing various options in the following sections. A vital part of the update process si the `smartmerge` python module. This module helps in merging JSON configuration files efficiently during the update, which helps transition from old version of the software to new without breaking/corrupting anything. `Smartmerge` also attempts to preserve your older configuration edits.

`Grahil installer` in autoupdate mode can seamlessly update the `grahil` program to latest without the need for any human intervention, very much like a high level operating system autoupdate.

## Requirements

The installer script use mix of `bash` ansd `python 3.7+` to install and update grahil. While bash is available everywhere, you must ensure your system can support `python 3.7 or higher`. The installer will scan the system looking for supported python versions and if not found it will look for them on `apt`/`yum` repositories.

* Min CPU Speed: 1 GHZ
* Min Memory: 256 MB
* CPU ARCH: 64/32 Bit Intel/AMD or 64/32 Bit ARM
* OS: RedHat, Ubuntu 16+, CentOS 7+

__**Coming soon: (Raspian, ARMbian, Ubuntu For ARM)**__

## How To Use

### Download the installer to your target system

* Clone the repository on your linux system using git client - `git clone https://github.com/connessionetech/grahil-installer.git`

* Execute the following command to switch directory to the installer script and make it executable - `cd grahil-installer && sudo chmod +x *.sh`

### Basic Usage (Simple Install)

* Run the script by typing - `./install.sh`.

> *NOTE: DO NOT USE `SUDO` TO RUN THE SCRIPT. IT WILL ASK FOR SUDO PERMISSIONS ON ITS OWN*

* The script will then request permission by prompting for superuser password. Enter password to alow the script to continue.

* If everything is fine the script will not prompt you for anything else. It will install the program and then register a systemd service for it automatically by the name `grahil.service`. The service is auto-started post installation. However you can still use `systemctl` to start and stop it manually.

Stop Service:

```bash
sudo systemctl stop grahil.service
```

Start Service:

```bash
sudo systemctl start grahil.service
```

### Arguments

| Flag  | Description  | Value Type   |   |   |
|---|---|---|---|---|
| -i | Run in install mode   | NA  |   |   |
| -u | Run in update mode   | (0 -> install or 1 -> update or -1 -> remove)  |   |   |
| -r | Run in uninstall mode (Targets core unless module flag is selected)   | NA  |   |   |
| -m | Sets module as subject of operations   | NA  |   |   |
| -h  | Usage information   | NA  |   |   |


## Functionality & Mechanisms

### The Manifest

The manifest is a JSON fornmatted file which is used to describe the installation of `grahil` core program. It defines various aspects of the installation such as supported python versions, release date, release version, package download url, package signature (md5 hash) etc. The installer will reade manifest and parse it before starting with the installation/updation mechanism.


```json

{
    "vendor": "Connessione Technoloigies Pvt. Ltd.",
    "released": "22-05-12 01:02:39",
    "payload": {
        "version": "1.0.18",
        "format": "zip",
        "platform": {
            "x86_64": {
                "enabled": true,
                "url": "https://*****************/core/x86_64/grahil.zip",
                "md5": "6dafac4c971e23b0beee60ce92a072f4",
                "dependencies": {
                    "interpreters": "3.7,3.8"
                },
                "cleanups": []
            },
            "arm64": {
                "enabled": false,
                "url": "",
                "md5": "",
                "dependencies": {
                    "interpreters": ""
                },
                "cleanups": []
            }
        }
    }
}

```

**Manifest can supports different builds for x86_64 & ARM platforms simultaneously**


* `vendor` - Name of manifest provider
* `released` - DateTime of manifest publish
* `payload.version` - Build version number.
* `payload.url` - Build download url
* `payload.format` - Package format
* `payload.md5` - MD5 hash signature of the build package
* `payload.dependencies.interpreters` - List of python interpreter versions supported by the build
* `cleanups` - List of files/folders to remove after update


### Basic Install Process

**COMMAND**

```bash
./install.sh
```

    or

```bash
./install.sh -i
```

From the installer's standpoint, the `installation` process can broadly be broken down into the following steps:

* Detect the system configuration, operating system and other system details
* Load variables from local configuration file (config.ini)
* Install all the prerequisites on the system
* Fetch and parse the build manifest from cloud
* Use information from manifest to discover or install a supported version of python on the system (if needed)
* Download the grahil payload & install to proper location
* Setup python virtual environment for the python version identified or installed earlier
* Create a systemd service for grahil using the main entry file & virtual environment interpreter

#### Modules

### Installing modules

```bash
./install.sh -i -m <module-name>
```

Where `-i` instructs to run script in installation mode & the `-m` flag selects module oprations.

`<module-name>` refers to the name of the module you are trying to install. the module should exist in the repository (should be supported officially). Custom module installation is not support via this installer at the moment.

__You need to restart grahil service for changes to take effect.__

### Removing modules

```bash
./install.sh -r -m <module-name>
```

Where `-r` instructs to run script in removal mode & the `-m` flag selects module oprations.

`<module-name>` refers to the name of the module you are trying to remove. If the module and its configuration exist in the install location it will be removed.

__You need to restart grahil service for changes to take effect.__

### Update Process

**COMMAND**

```bash
./install.sh -u 1
```

From the installer's standpoint, the `update` process can broadly be broken down into the following steps:

* Detect the current instalaltion and configurations
* Fetch and parse the build manifestfrom cloud
* Determine whether installation is updatable or not
* Download the payload to disk
* Use Smartmerge to merge old and new configuration files properly to prepare the latest payload for deploymenmmt.
* Check & create python virtual environment as needed.
* Install dependencies for latest build 
* Update the systemd service if necessary

#### SmartMerge

`Smartmerge` is a python program that is used to merge configuration files from existing version of the software and the latest version of the software downloaded, without losing the edited configurations. Smartmerge script reuses the grahil virtual environment for its dependencies and hence does not require setting up a new python virtual environment.

#### AutoUpdater (experimental)

Autoupdate is a useful (experimental) feature of this installer which lets you update your existing grahil installation automatically in an unattended manner.

To activate autoupdater :>

1. Make sure the install.sh script has administrative rights and permissions to execute.
2. Create a CRON job in the linux system to run the bash script as the administrator automatically once a day at a specific time.

The cron job would look like this :

```bash

TO DO

```


### Uninstall Process

**COMMAND**

```bash
./install.sh -r
```

    or

```bash
./install.sh -u -1
```

From the installer's standpoint, the `uninstall` process can be broken down into the following steps:

* Check system for existing installation
* Remove grahil core & modules
* Remove any virtual environment created
* Remove any systemd service created
* Remove all installation data

## Future Roadmap

* Custom manifest support for custom installation
* Enable/disable module
* Custom dependencies injection during installation
