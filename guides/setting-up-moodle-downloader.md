---
title: Moodle Downloader Setup for Dummies
description: An unofficial guide on how to set up the MoodleDownloader2 with Docker and a cronjob.
published: false
date: 2022-12-25T13:05:57.907Z
tags: guide, archive
editor: markdown
dateCreated: 2022-12-24T16:45:56.119Z
---

# Moodle Downloader Setup for Dummies

In this guide, I'll walk you through setting up the [C0D3D3V/Moodle-Downloader-2](https://github.com/C0D3D3V/Moodle-Downloader-2) repository on Linux. In this example I'll be using Ubuntu, but you should be able to follow along with different Debian-based Distros. The install instructions might be different for some of the dependencies.

## Prerequisites

You'll need some kind of of linux shell in front of you. If you're using Ubuntu Desktop, just open the terminal.

Docker is used to run the moodle downloader in a container with all the dependencies pre-installed.
Install it using the [Docker apt repositories](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)^[You can also install Docker Desktop, but I recommend against it]. You can check your installation using `sudo docker version`.

## Building the Docker Image

In this step, we will be creating a container image from the source code, because no pre-built image is provided by the repository [yet](https://github.com/C0D3D3V/Moodle-Downloader-2/issues/171).

First, you'll need to download the source code. Either download the ZIP file from GitHub and extract it, or if you want to be more easily be able to update the image later, use 
```
~ $ git clone https://github.com/C0D3D3V/Moodle-Downloader-2.git
```
This requires `git`, which you can install using `sudo apt install git`.


To build the image, use the `docker build` command to create an image from the source code. The following command will work if you used `git clone` otherwise you'll have to adjust the path (last argument) to point to the source code.

```
$ sudo docker build -t moodle-dl:latest Moodle-Downloader-2
```

Any time you want to update your local image, you will have to download the new source code, against either as a zip, or by using `git pull` inside the folder that contains the source code, if you used git to download the source code. Then, rebuild the image using the same command.

## Configuration

Now that you have the container ready, you can use it to run moodle downloader commands. Create the directory where you want your moodle and configuration files to reside, then cd into it:

```
~ $ mkdir moodle
~ $ cd moodle
```

For initial configuration, you can use the Moodle Downloader configuration wizard. I recommend going through the advanced configuration and reading through the options and choosing the "Blacklist" mode when prompted, so that new courses will be downloaded automatically.

```
~/moodle $ sudo docker run --rm -it --net=host -v $(pwd):/files moodle-dl --init
Do you want to activate Notifications via mail? (y/N) 
Do you want to activate Notifications via Telegram? (y/N) 
Do you want to activate Notifications via XMPP? (y/N) 
Do you want to configure Error Reporting via Sentry? (y/N) 
[The following Credentials are not saved, it is only used temporarily to generate a login token.]
URL of Moodle:   https://moodle.some-university.de/
Username for Moodle:   *********
Password for Moodle [no output]:   
Configuration finished and saved!
[...]
You can always do the additional configuration later with the --config option.
Do you want to make additional configurations now? (y/N) Yes
[...]
Configuration successfully updated!
All set and ready to go!
```

Note that your login information is not saved directly, however a token which gives full access to your moodle account is stored. If either the 'config.json' or the 'Cookies.txt' is obtained by a malicious third party, they might have full access to your moodle account. So guard these files appropriately.

## Downloading the Moodle data

Downloading all files from all configured courses is very straightforward:

```
~/moodle $ docker run --rm -it --net=host -v $(pwd):/files moodle-dl --ignore-ytdl-errors
[...]
All done. Exiting..
```

## Syncing with the cloud using rclone

This step is optional, but I highly recommend it, unless you have some other back up strategy in place already.

We'll be using `rclone`, a very popular file manager for cloud storages to sync our files. Install rclone using `sudo apt install rclone`, you can check your installation using `rclone version`.

Students in Germany get very cheap OneDrive Storage (https://bildung365.de). Follow the offical rclone guide on how to set up a [OneDrive remote](https://rclone.org/onedrive/) or any other cloud provider.

After you have configured your remote, add a folder where you would like to store the moodle data:

```
~/moodle $ rclone mkdir onedrive:path/to/remote/moodle
```

Now you can upload your local folder to the remote using the `rclone copy` command. Add the exclude parameter if you don't want your moodle tokens to end up in the cloud.

```
~/moodle $ rclone copy . onedrive:path/to/remote/moodle --exclude config.json --exclude Cookies.txt
```

## Cronjob

To regulary run and upload your files, you can create a small bash script:

```bash
#!/bin/bash
rm -f Cookies.txt
docker run --rm -v $(pwd):/files moodle-dl --ignore-ytdl-errors
rclone copy . onedrive:path/to/remote/moodle --exclude Cookies.txt --exclude config.json
```

Save this as `dump_and_upload.sh` inside your moodle directory.

Now, mark the script executable and run it:

```
~/moodle $ chmod +x dump_and_upload.sh
~/moodle $ ./dump_and_upload.sh
[...]
```

Once the script is done, the files should have been uploaded to your remote.

To run this script every day at 3am, you can set up a cronjob. If it's your first time running `crontab -e`, you might need to choose an editor.

```
~/moodle $ $ crontab -e

no crontab for user - using an empty one
Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed
  
 Choose 1-4 [1]: 1
```

Add the following line to your crontab, replacing `/home/user/moodle` with the full path to your moodle directory.

```crontab
0 3 * * * (cd /home/user/moodle/ && ./dump_and_upload.sh)
```

Exit the editor to save your changes.


## Renewing credentials

This should not be neccesarry, but if you're having trouble with expiring tokens, then you might want to add a new line to the `dump_and_upload.sh` script which refreshes your token every time it is run:

```
#!/bin/bash
rm -f Cookies.txt
docker run --rm -v $(pwd):/files moodle-dl --new-token --username XYZ --password XYZ
docker run --rm -v $(pwd):/files moodle-dl --ignore-ytdl-errors
rclone copy . onedrive:path/to/remote/moodle --exclude Cookies.txt --exclude config.json
```

This will require you to save your password in plain text inside the dump_and_upload.sh script, so consider the security implications.


## Summary

You should now have a working setup which dumps your moodle courses and saves them to your cloud storage provider once a day. 

Here are all commands summarized again:

```
~$ docker version
[..]
~ $ git clone https://github.com/C0D3D3V/Moodle-Downloader-2.git
[...]
Receiving objects: 100% (4100/4100), 907.44 KiB | 7.00 KiB/s, done.
Resolving deltas: 100% (2902/2902), done.
~ $ sudo docker build -t moodle-dl:latest Moodle-Downloader-2
~ $ mkdir moodle
~ $ cd moodle
~/moodle $ sudo docker run --rm -it --net=host -v $(pwd):/files moodle-dl --init
~/moodle $ sudo docker run --rm -it --net=host -v $(pwd):/files moodle-dl --ignore-ytdl-errors
~/moodle $ rclone copy . onedrive:path/to/remote/moodle --exclude Cookies.txt --exclude config.json 
~/moodle $ cat > dump_and_upload.sh <<EOF
#!/bin/bash
rm -f Cookies.txt
docker run --rm -v $(pwd):/files moodle-dl --ignore-ytdl-errors
rclone copy . onedrive:path/to/remote/moodle --exclude Cookies.txt --exclude config.json
EOF
~/moodle $ chmod +x dump_and_upload.sh
~/moodle $ # Now, add the dump_and_upload.sh to a cronjob (crontab -e)
```