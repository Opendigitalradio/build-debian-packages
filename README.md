# Introduction
This repository contains a github action whose goal is to:
1. Build the debian packages for 2 suites (`bookworm` and `bullseye` as of July 2023) and 3 architectures (`amd64`, `arm64` and `armhf` as of July 2023)
1. Push the debian packages to the [Opendigitalradio debian apt repository](http://debian.opendigitalradio.org)

# Repository setup
Your repository must contain the following:
1. Secret `DEBIAN_GIT_TOKEN` storing the token to checkout the debianized sources git repository
1. Secret `APT_SSH_KEY` storing the ssh key to access the apt server
1. Variable `DEBIAN_GIT_SERVER` storing the url of the git server hosting the debianized sources
1. Variable `DEBIAN_GIT_OWNER` storing the owner of the debianized sources on the git server
1. Variable `APT_SSH_TO` storing the ssh destination where the debian packages will be sent to
1. Variable `APT_SSH_KNOWN_HOSTS` storing the output of the command `ssh-keyscan -p 222 -t ed25519 mpb.li`

# Operations
The debian packages should be built and pushed only after a new version is created on the debian git repository

## Build and push a debian package
1. Select the menu `actions` on the github web interface
1. Select the `Build And Push Debian Packages` action on the left
1. Click on the `Run workflow` button on the right
1. Select the Opendigitalradio tool
1. Click on the `Run workflow` button and relax

Once the action is completed, the debian package files are available in the directory `/home/robinalexander/incoming/{bookworm,bullseye}` folders

## Update the apt repository
You will need to have:
1. read/write access to the `incoming` folder on the apt server
1. the GPG private key to sign the debian packages when running the `reprepro` command below

Run the following commands on your local host (not the apt server):
1. Sync the **incoming** folder to your localhost from the apt server
   ```
	 rsync \
	   --archive \
		 --recursive \
		 --rsh="ssh -p 222" \
		 robinalexander@mpb.li:incoming .
   ```
1. Sync the **apt** folder to your local host from the apt server
   ```
	 rsync \
	   --archive \
		 --recursive \
		 --rsh="ssh -p 222" \
		 robinalexander@mpb.li:apt .
   ```
1. Include the debian packages on your local host
   ```
	 for suite in bookworm bullseye; do
	   reprepro include "${suite}" <debian_package>.changes
	 done
   ```
1. Sync the **apt** folder to the **apt server** from your local host
   ```
	 rsync \
	   --archive \
		 --recursive \
		 --rsh="ssh -p 222" \
		 apt robinalexander@mpb.li:
   ```
