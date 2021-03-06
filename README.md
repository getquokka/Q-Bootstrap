# Q-Bootstrap

This repository contains the necessary images for a raspbian to be bootstrapped with customized software on it.
The idea is to build a very simple buildroot, aimed at raspbian but it should work with any linux based image.

Key features are:
* Specifically targeted at Raspbian
* Extremely fast
* Reproducible builds
* Very easy learning curve
* Basic but flexible
* Full build performed every time
* No dependencies (or, at least, you should be able to `apt install` everything from an Ubuntu or debian)

## Project overview

The project is structured into various directories:
* tools: Q-Bootstrap files and tools. All scripts you run will be here.
* skel: A skeleton directory structure, which you can copy and modify for your own customizations

## Skeleton Directory structure

Take a look at the `skel` directory. You can `cp -r skel your_test` to create your own customized image. Everything needed to build an image is in the skel directory.

Structure:
* `config.env`: The most important file: This controls all aspects of your build
* `img`: the images to build are stored in this directory. You can either download and extract them yourself or specify a configuration file
* `build`: output files generated by Q-Bootstrap
* `overlays`: Files in overlays/* are copied to the output image as is
* `scripts`: scripts to be run when the image is still mounted. run-parts will be used to run them in order


You will be working with a "SRC_DIR" directory where your customizations go. in your SRC_DIR, you must have:


Your skeleton can be anywhere. the `examples` folder illustrates a simple case of a Raspbian


## Dependencies

We need some packages to do our magic

```
sudo apt install -y whois debootstrap binfmt-support cloud-guest-utils
sudo apt install -y qemu qemu-user-static
# let me know if i forget something here
```

## Environment

`config.env` files are small scripts to be sourced by the build script.

They configure various aspects of your build. See `skel/config.env` for an example.


## Preparing the build

You can build an image following the following steps and customization options:

* Clone the skel directory to your own private build

```
cp -r skel priv
```

* Download and extract a raspbian image into the `img` folder
* Edit your `config.env`
  * Set `Q_IN_IMG` to `$Q_IMG_DIR/<your_new_image>`
  * Set `Q_PKG_LIST` to any packages you want to install with `apt` during
    image generation. (`Q_PKG_LIST="screen git python3"`)
  * If you are going to be iterating a lot of images and want to autoflash onto a sd card:
    Set `Q_AUTOFLASH_DRIVE=/dev/sdXXX` to your SDcard (no partitions, just the card)
  * Set `Q_KEEP_MOUNTED` to true if you want to explore the resulting filesystem before unmounting
  
  
### Add *Overlays*

Overlays are just files that will overwrite anything on
the pi's original filesystems. You can use `overlays/1` for the boot partition
and `overlays/2` for the ext4 rootfs. 

* A good example is to enable ssh:
```
touch `priv/overlays/1/ssh
```

* And to add your SSH key to login to the pi
  
```
keys_file="priv/overlays/2/home/pi/.ssh/authorized_keys"
mkdir -p $(dirname $keys_file)
cat ~/.ssh/id_rsa.pub > $keys_file
chmod 600 $keys_file
sudo chown 1000:1000 $keys_file
```

* Maybe you want your pi to auto-connect to your wifi:

```
cat << EOF > priv/overlays/2/etc/wpa_supplicant/wpa_supplicant.conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
	ssid="Your WIFI SSID"
	psk="Your WIFI password"
}
EOF
```


### Run *Scripts*
You can run scripts too, just place them in the `priv/scripts` directory
  * Scripts are run in alphabetical order
  * Interpreted by /bin/bash in the chrooted environment of the sdcard
  * nonzero exit status will finish the build process

Note that you should not set hostname with the hostname command in this script directly, instead, modify the files /etc/hostname and /etc/hosts in overlays
  

### Build the image
Run the build script to generate the image
```
# From the Q-Bootstrap directory:
tools/build.sh priv/config.env
```

You can see the skel directory for an example (both overlays and scripts are explained in detail)


