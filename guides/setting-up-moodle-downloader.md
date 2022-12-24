---
title: Moodle Downloader Setup for Dummies
description: An unofficial guide on how to set up the MoodleDownloader2 with Docker and a cronjob.
published: false
date: 2022-12-24T16:45:56.119Z
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

For initial configuration, you can use the Moodle Downloader configuration wizard:

```
~/moodle $ docker run --rm -it --net=host -v $(pwd):/files moodle-dl --init
TODO!!
```

> TODO! ^
{.is-danger}

Note that your login information is not saved directly, however a token which gives full access to your moodle account is stored. If either the 'config.json' or the 'Cookies.txt' is obtained by a malicious third party, they might have full access to your moodle account. So guard these files appropriately.

Hint: If you want to add all courses visible to your user account, you can use `docker run --rm -it --net=host -v $(pwd):/files moodle-dl --add-all-visible-courses`.

## Downloading the Moodle data

Downloading all files from all configured courses is very straightforward:

```
~/moodle $ docker run --rm -it --net=host -v $(pwd):/files moodle-dl --ignore-ytdl-errors
```

## Syncing with the cloud using rclone

This step is optional, but I highly recommend it, unless you have some other back up strategy in place already.

We'll be using `rclone`, a very popular file manager for cloud storages to sync our files. Install rclone using `sudo apt install rclone`, you can check your installation using 

Students in Germany get very cheap OneDrive Storage (https://bildung365.de), so I'll be using OneDrive in this example, but any of rclone's supported targets will of course work.

> TODO
{.is-danger}


## Summary

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
~/moodle $ docker run --rm -it --net=host -v $(pwd):/files moodle-dl --init
~/moodle $ docker run --rm -it --net=host -v $(pwd):/files moodle-dl --add-all-visible-courses
~/moodle $ docker run --rm -it --net=host -v $(pwd):/files moodle-dl --ignore-ytdl-errors
~/moodle $ rclone copy . onedrive:example/target/folder --exclude Cookies.txt --exclude config.json 
```
