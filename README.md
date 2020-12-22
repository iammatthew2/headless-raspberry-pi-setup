# Setting up a headless Raspberry Pi for Node.js development

This incomplete guide walks through setting up a development environment on a headless Raspberry Pi. It leverages [watchman-processor](https://github.com/markis/watchman-processor) to sync code from a primary machine to the Raspberry Pi. To run processes and tweak configs we will ssh into the Pi and execute commands directly on the Pi. And we make use of [Tmux](https://medium.com/@tholex/what-is-tmux-and-why-would-you-want-it-for-frontend-development-e43e8f370ef2) _(optional)_ to provide a persistent ssh session.

## Requirements

* A Raspberry Pi (a Pi 3+ was tested on this flow. **Note**: if using a Zero a few parts of this guide are going to be problematic: Node development is probably going to be quite slow and installing Node will require a different approach than the one described below in this guide; plugging an ethernet cable into a Zero is going to require some magic)
* An Ethernet connection
* Admin access to your router so we can get your Pi's IP address (**required** since this is a headless setup)
* Wifi connection _(optional)_

## Setup the Pi

* Prepare the Raspberry Pi image on your SD card (or just use the Raspberry Pi imager they now provide):
  * Download the latest image (use the lite image since this is a headless setup): <https://www.raspberrypi.org/downloads/>
  * Burn the image to your SD card using <https://etcher.io/> or something similar
* Enable ssh: place a file named “ssh” onto the boot partition of the SD card
  * For more info see [Enable SSH on a headless Raspberry Pi](https://www.raspberrypi.org/documentation/remote-access/ssh/)
* Setup wifi:
  * Insert the SD card into the Pi, connect the Pi to your network via the an ethernet connection and power up the Pi
  * Grab the ip address by logging into your router admin panel. You may need to [Google for help](https://www.google.com/search?q=tp%20link+admin+login) logging in to your router
  * Now ssh into the Raspberry pi: `$ ssh pi@<ip_address>`
  * You should be prompted for the password. The default password is `raspberry`
  * Update the default password: `$ passwd`
  * Add the wifi credentials to the pi:
    * Open up the config file: `$ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`
    * And update the settings with the following:

    ```bash
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    network={
      ssid="<wifiNetworkName>"
      psk="<wifiPassword>"
      key_mgmt=WPA-PSK
    }
    ```

  * After the wifi credentials are successfully saved unplug the ethernet cable and reboot the Pi (`$ sudo reboot now`)
  * When the Pi is booted up again you will probably have a new ip address. Go back and check your router admin panel to check:
  
    1. did the Pi successfully connect to your network over wifi?
    2. if it is connected, grab the IP address and ssh again into the Pi: `$ ssh pi@<ip_address>`
  
  * If you can ssh in from wifi, you are done with this part of the setup. If the Pi did not reconnect, it's time to start Googling :(

* Update the Pi and install vim, git and tmux:
  * `$ sudo apt-get update`
  * `$ sudo apt-get node npm install vim git tmux mosh`
* Add the new source, then install node and npm (a Raspberry Pi Zero will have trouble with this next step):
  * `$ curl -sL https://deb.nodesource.com/setup_.x | sudo -E bash` ## note: update to latest version
  * `$ sudo apt-get install nodejs`
  * Now install yarn:

  ```bash
  curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list && sudo apt-get update && sudo apt-get install yarn
  ```

* *Optional*: Add keys for password-free login
  * [Here is a guide for ssh-key setup](https://www.raspberrypi.org/documentation/remote-access/ssh/passwordless.md), but if you already have keys then you just need to copy them over to the Pi. If your main machine has `ssh-copy-id` then all you need to do is `$ ssh-copy-id -i <id> pi@<ip>` () and you should be good to go
  * Once your keys are setup, you should be able to `$ ssh pi@<ipAddress>` and not be prompted for a password
* *Optional*: Rename the Pi's host name to something unique to keep track of which pi you are logged into. To do so, update the name in these two files:
  * `$ sudo vim /etc/hostname`
  * `$ sudo vim /etc/hosts`
* *Optional*: Setup internal reserved ip address: this would be a router-specific set of steps, but the gist of it is that you would grab your Pi's MAC address (easily done in most router admin panels) and then find the router's address reservation settings where you will add your desired static internal IP and associate that with your device via the MAC address. With that done, add that ip address to you ssh config file settings. Now logging in will look like `$ ssh <hostName>`
* *Optional*: Setup your [dotfiles](https://dotfiles.github.io/) if you have some - or feel free to use [mine](https://github.com/iammatthew2/dotfiles)
