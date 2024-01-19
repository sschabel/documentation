# Jellyfin Docker Setup on Raspberry Pi
## Contents
 - [Overview](#overview)
 - [Prepping your machine for Jellyfin](#prepping-your-machine-for-jellyfin) 
 - [Mounting an external drive containing media](#mounting-an-external-drive-containing-media)
 - [Starting Jellyfin using Docker Compose](#starting-jellyfin-using-docker-compose)
## Overview
This is simply documenting how I setup Jellyfin on my Raspberry Pi so I can reference it again if needed .Feel free to use this if you want. A lot fo this is already in the Jellyfin documentation.

## Prepping your machine for Jellyfin
Follow the below steps to prep your Jellyfin host machine.

1. Add a new user named ``jellyfin`` and make the user's home ``/opt/jellyfin``
    ```sh
    sudo useradd -m -r -d /opt/jellyfin jellyfin
    ```
2. Run the following commands to create the necessary directories for Jellyfin to use for its configuration, cache, and media.
> [!NOTE]
> You can use an external drive for storing Jellyfin media. See the [Mounting an external drive containing media](#mounting-an-external-drive-containing-media) section below.

    ```sh
    sudo mkdir /opt/jellyfin/config
    sudo mkdir /opt/jellyfin/cache
    sudo mkdir /opt/jellyfin/media
    ```
3. Assign the owner of the directories you just made to the new ``jellyfin`` user:
    ```sh
    sudo chown -R jellyfin /opt/jellyfin/config/
    sudo chown -R jellyfin /opt/jellyfin/cache/
    sudo chown -R jellyfin /opt/jellyfin/media/
    ```
4. If you haven't already, install Docker.
5. Get the Jellyfin Docker image:
    ```sh
    docker pull jellyfin/jellyfin
    ```
## Mounting an external drive containing media
Following the below steps to mount your external drive and setup Jellyfin to use it for media. The below steps were taken from here: https://www.wundertech.net/how-to-setup-jellyfin-on-a-raspberry-pi/

1. Create the media mount for your external drive to be mounted to:
    ```sh
    sudo mkdir /mnt/media
    ```
2. Get the list of drives attached to your machine by running the following:
    ```sh
    sudo fdisk -l
    ```
3. Run the following command to get the UUID of the drive you want to mount:
    ```sh
    sudo blkid
    ```
4. Open up your favorite text editor to modify ``/etc/fstab``. You will need to save it using ``sudo``:
    ```sh
    code /etc/fstab
    ```
5. Add the following line to the /etc/fstab
    ```
    UUID=[the UUID of your external hard drive] /mnt/media [drive type] defaults,nofail 0 2
    ```
6. Edit the Docker compose.yaml file to reference the ``/mnt/media`` directory for the ``:media`` volume.

## Starting Jellyfin using Docker Compose
1. Create a new file called ``compose.yaml``
2. Open up ``compose.yaml`` and put the following content with the appropriate info filled in:
    ```yaml
    version: '3.5'
    services:
    jellyfin:
        image: jellyfin/jellyfin
        container_name: jellyfin
        user: [UID]:[GUID]
        network_mode: 'host'
        volumes:
        - /opt/jellyfin/config:/config
        - /opt/jellyfin/cache:/cache
        - [YOUR-MEDIA-LOCATION]:/media
        restart: 'unless-stopped'
    ```
3. After you have filled in the necessary data in the ``compose.yaml``, run the following command in the same directory as the ``compose.yaml``. This will run the Docker container in the background.
    ```sh
    docker compose up -d
    ```