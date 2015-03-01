# Raspberry Pi Dramble

A cluster ([Bramble](http://elinux.org/Bramble)) of Raspberry Pis on which Drupal will be deployed using Ansible.

Here's the overall architecture of the Dramble:

<img src="https://raw.githubusercontent.com/geerlingguy/raspberry-pi-dramble/master/images/raspberry-pi-dramble-lemp-redis-architecture.png" alt="Raspberry Pi Dramble - Server Architecture Diagram" />

## Why

I'm doing presentations on Ansible, and how easy it makes infrastructure configuration, even for high-performance/high-availability Drupal sites. WiFi/Internet access is spotty at most conferences, so deploying to AWS, DigitalOcean, or other live public cloud instances that require a stable Internet connection is a Bad Idea™.

Deploying to VMs on my own presentation laptop is an option (and I've done this in the past), but it's not quite as impactful as deploying to real, live, 'in-the-flesh' servers. Especially if you can say you're carrying around a datacenter in your bag!

A cluster of servers, in my hand, at the presentation. With blinking LEDs!

## Specs

  - 24 ARMv7 CPU Cores
  - 5.4 GHz combined compute power
  - 6 GB RAM
  - 96 GB microSD-based storage
  - 1 Gbps private network

As you can see from the above specs; it takes 5+ Raspberry Pis to equal the capacity of one moderate workstation nowadays. Raspberry Pis are not the most economical or power-saving way to build Drupal infrastructure, but they are great for educational purposes, and it's a fun project to build 'bare metal' infrastructure with your own hands!

## Getting the Pis (and other accessories)

For the Dramble, I purchased the following:

  - 6x Raspberry Pi 2 Model B
  - 1x [50W 6-port USB desktop charger](http://www.amazon.com/gp/product/B00KHP6UVQ/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B00KHP6UVQ&linkCode=as2&tag=httpwwwmidw06-20&linkId=YEKQEOUTP3WTLSJJ) (to provide juice to the Pis)
  - 1x [6-pack micro USB cables](http://www.amazon.com/gp/product/B00N8VHW72/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B00N8VHW72&linkCode=as2&tag=httpwwwmidw06-20&linkId=63VSGWYRPJFO4IZO) (plugs from charger to Pis - cheap in bulk!)
  - 1x [5-pack 8GB microSD cards](http://www.amazon.com/gp/product/B00KI16OOW/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B00KI16OOW&linkCode=as2&tag=httpwwwmidw06-20&linkId=JM2T4CPMOOJA44AW) (one for each Pi, cheap in bulk!)
    - Note that I'm actually using a hodgepodge of different microSD cards for different servers, as I've found pretty wide performance gaps in real-world usage on the Pi. My current top contender for price-performance is the SanDisk Extreme microSD card.
  - 1x [8-port unmanaged Gigabit network switch](http://www.amazon.com/gp/product/B001QUA6RA/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B001QUA6RA&linkCode=as2&tag=httpwwwmidw06-20&linkId=24SPP5YZJR6KK7GH) (for inter-Pi networking)
  - 1x [Raspberry Pi B+ Stackable Case Triplestack](http://www.ebay.com/itm/271648357906)
  - 2x [Raspberry Pi B+ Case, Stackable - Additional Level](http://www.ebay.com/itm/271614825269)

Other necessities, which I already had on-hand (but you might need to purchase):

  - Cat 5e or Cat 6 network cable for making patch cords. (I have hundreds of feet of the stuff, and can quickly punch down a patch cable, so I just made my own).
  - A power outlet. (I am planning on adding an inline UPS at some point in the future, and could run the cluster off this for a few minutes at least... so the power outlet is somewhat optional :)
  - A computer to run the Ansible playbooks (you can do it all on the Pis themselves, but I prefer working on my Mac workstation and leaving all the Pis headless).

Other accessories, optional but helpful for general Pi development:

  - [802.11b/g/n USB WiFi adapter](http://www.amazon.com/gp/product/B003MTTJOY/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B003MTTJOY&linkCode=as2&tag=httpwwwmidw06-20&linkId=WCTEZWNBLNNQ35E5) (I have a few; handy for getting online away from wired jack)
  - [5mm Diffused RGB LEDs](http://www.amazon.com/gp/product/B006S21SAK/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B006S21SAK&linkCode=as2&tag=httpwwwmidw06-20&linkId=2D7X6HRTJFTESGT2) (great for showing status!)
  - [Assembled RPi GPIO Cobbler with 40-pin cable](http://www.amazon.com/gp/product/B00Q1T07O8/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=B00Q1T07O8&linkCode=as2&tag=httpwwwmidw06-20&linkId=JG3OGOMBG75D6BFY) (helpful for prototyping)
  - Half-size breadboard
  - Breadboard/prototyping jumper cables

## Setting up the Pis

### Preparing the microSD cards (with Raspbian)

Download Raspbian from the [Raspberry Pi Operating System Images](http://www.raspberrypi.org/downloads/) page, and expand the .zip file after it's finished downloading.

You could copy this image file to each of your microSD cards directly using `diskutil unmountDisk /dev/diskX` and `sudo dd bs=1m if=/path/to/2015-01-31-raspbian.img of=/dev/diskX` (where `X` is the number for the microSD card on your Mac - found with `diskutil list`), however, I recommend using [`diet-raspbian`](https://github.com/geerlingguy/diet-raspbian) to build a minimal image based off Raspbian.

[`diet-raspbian`](https://github.com/geerlingguy/diet-raspbian) is included as a git submodule inside the `setup` folder, but you should read the instructions included in that project's [README](https://github.com/geerlingguy/diet-raspbian/blob/master/README.md).

The basic process is as follows:

  1. Clone the official Raspbian image to a microSD card and boot a Pi with it.
  2. Trim the fat from the distribution using the included `diet.yml` Ansible playbook.
  3. Resize the partition to a much smaller size (from ~3GB to ~1.25GB).
  4. Clone the new, minified Raspbian image to your Mac.
  5. Copy this new, minified image to all your microSD cards.

`diet-raspbian`'s instructions are very thorough and it should work with the latest build of Raspbian.

### Preparing the Raspberry Pis

Assuming you've cloned a `diet-raspbian`-based image, you shouldn't need to configure the Pis directly (via terminal session with a connected display/keyboard). However, if you use the default Raspbian image, and still need to perform the initial configuration steps (e.g. `raspi-config`), you can still do so headless via SSH:

  1. Find the Raspberry Pi's IP address, and connect via SSH: `ssh pi@[IP-ADDRESS]`
  2. Default username is `pi` and default password is `raspberry`.
  3. Once logged in, run `sudo raspi-config`, and follow the prompts.

Even if you *did* use the `diet-raspbian` image, you may wish to expand the filesystem to fill the entire microSD card on each Pi using the first option in `sudo raspi-config`. We'll go through these steps in detail later.

### Racking the Raspberry Pis

  1. Mount one Pi per tier of the stackable case.
  2. Mount the stack to a board, and mount the network switch and USB power supply as well.
  3. Plug each Pi into one of the power supply's jacks, ensuring any Pis that will be powering extra USB devices (e.g. the database server) is plugged into a 2A port.
  4. Create custom-sized Cat 5e/Cat 6 cables, one-per-Pi, to connect to the network switch.
  5. Run power to the switch and to the USB power supply.
  6. Profit!

### Provisioning the Raspberry Pis

Once all the Pis are booted, and you are able to log into them over the network via SSH, there are a few steps you need to perform before you'll be ready to run the `provision.yml` playbook to provision software to them.

#### Finish Pi configuration with `raspi-config`

The first thing you need to do is finish basic Pi configuration for each Pi:

  1. Log into the Pi directly or via SSH, and run `sudo raspi-config`.
  2. Select 'Expand Filesystem' and then Ok when it's complete.
  3. Select 'Change User Password' if you would like to set a different password for the 'pi' account.
  4. Set Overclocking options if you so desire.
  5. Select 'Finish' and restart the Pi so the filesystem is expanded.

> You may also need to run `sudo swapoff --all && sudo rm -rf /var/swap` if you get an 'out of disk space' error in any of the preceding steps.

#### Setting up networking

There is an included playbook (inside `setup/networking`) which will set up all the Pi networking configuration following the below network layout:

  - `bal1.pidramble.com` (10.0.1.60)
  - `www1.pidramble.com` (10.0.1.61)
  - `www2.pidramble.com` (10.0.1.62)
  - `www3.pidramble.com` (10.0.1.63)
  - `cache1.pidramble.com` (10.0.1.64)
  - `db1.pidramble.com` (10.0.1.65)

To use it, you will need to know the IP addresses and MAC addresses for all six Pis as they are currently set up. Map each MAC address to the new structures inside the networking `vars.yml`, and add all the Pi IP addresses under the `[pis]` group inside the networking `inventory` file. Then run (within the `setup/networking` directory):

    $ ansible-playbook -i inventory main.yml

After this, you will need to reboot all the Pis, which can be done from inside the same directory (since those IP addresses will still be active until after the reboot) using:

    $ ansible -i inventory -a "shutdown -r now" -s

Once rebooted, you should be able to follow the next steps to set up the Pis.

> This networking configuration works for me and my local network setup. If you'd like to adjust the private network range used, or the IP addresses, etc., you'll need to do that on your own :)

#### Testing Ansible configuration

Once you have all your Pis configured, and the `inventory` file has all the right IP addresses, you can run the following command (within this directory) to test Ansible's ability to see all the Pis:

    $ ansible all -i inventory -m ping

This should return something like:

    $ ansible all -i inventory -m ping
    10.0.1.60 | success >> {
        "changed": false,
        "ping": "pong"
    }
    
    ...
    
    10.0.1.65 | success >> {
        "changed": false,
        "ping": "pong"
    }

If you get a `pong` for each Pi, you're good to go!

Some other interesting Ansible commands you could run to manage your Pis:

    # Get current disk usage.
    $ ansible all -i inventory -a "df -h"
    
    # Get current memory usage.
    $ ansible all -i inventory -a "free -m"
    
    # Reboot all the Pis.
    $ ansible all -i inventory -a "shutdown -r now" -s
    
    # Shut down all the Pis.
    $ ansible all -i inventory -a "shutdown -h now" -s

#### Running `main.yml`

Once all the Pis are online and operational, all you need to do to get them provisioned is run the command below (in the same directory as this README), then sit back for a few minutes while all the software is installed and configured (note that you may need to run `ansible-galaxy -r playbooks/requirements.txt` to install Ansible role dependencies if they're not already on your system):

    $ ansible-playbook -i inventory main.yml

After 5-10 minutes, you should see a summary of the completed tasks:

    PLAY RECAP ****************************************************************
    10.0.1.60               : ok=4   changed=7    unreachable=0    failed=0
    10.0.1.61               : ok=3   changed=8    unreachable=0    failed=0
    10.0.1.62               : ok=3   changed=8    unreachable=0    failed=0
    10.0.1.63               : ok=3   changed=8    unreachable=0    failed=0
    10.0.1.64               : ok=3   changed=5    unreachable=0    failed=0
    10.0.1.65               : ok=4   changed=9    unreachable=0    failed=0

At this point, all the software that will support your Drupal site is installed. The next step is to deploy Drupal to the infrastructure.

> To un-provision a Pi, you can either use `apt-get remove --purge [package]` and `apt-get autoremove` to remove installed packages/configuration, then reboot... or you can reimage the microSD cards from a fresh copy of Raspbian or `diet-raspbian`. I prefer the latter, but for quick testing/experimentation will do the former (or create Ansible playbooks to back out all the changes).

#### Testing the default balancer configuration

At this point, the infrastructure should already be serving http requests through the load balancer, and you can verify this by opening your browser and pointing it at the IP address of your balancer (in my case, `http://10.0.1.60/`).

Refresh the page a few times (if you're using Chrome, use Shift + Command + R to refresh the cached page), and you should see the IP address on the page switch between the three different webserver IP addresses.

### Deploying Drupal to the Raspberry Pis

First, before we deploy our Drupal 8 site to the Raspberry Pis, you should set up your local workstation's hosts file so you can enter the web address `http://pidramble.com/` in your browser and load the site through the load balancer. You will need to add the following line to the end of your hosts file:

    10.0.1.60  pidramble.com

Instructions: [editing your hosts file](http://www.rackspace.com/knowledge_center/article/how-do-i-modify-my-hosts-file).

Once you save the new version of the hosts file, go visit `http://pidramble.com/`. You should get a blank page that says "File not found." This means Nginx is ready to serve up your site, but the Drupal site isn't yet present.

#### Perform the initial Drupal deployment and installation

Deploy Drupal to the Dramble by running the following command (in this directory):

    $ ansible-playbook -i inventory playbooks/drupal/main.yml

Once the deployment is complete, you should be able to reload `http://pidramble.com/`, and see a pretty generic looking Drupal 8 installation, with a few links and a login screen (this is Drupal 8's default minimal installation look and feel).

#### Deploy an update to the Drupal 8 site

Let's assume that some time has passed, and you've done a bunch of great work on your Drupal site, and you've committed configuration changes and new code to the `demo-drupal-8` codebase. The initial version of the repository that was installed was 0.9.8, but now you have a slick new 1.0.0 release.

To deploy this update, run the same command as above, but pass in the new `drupal_version` override using Ansible's `--extra-vars` command line option:

    $ ansible-playbook -i inventory playbooks/drupal/main.yml --extra-vars "drupal_version=1.0.0"

Reload `http://pidramble.com/` again, and this time, you should see the site looking a lot nicer, using Drupal's default Bartik theme. If you log in, you'll notice the configuration in 1.0.0 of the `demo-drupal-8` project has been applied, and the site should be ready to use for simple content management.

#### Enabling Redis cache backend

After the update to `drupal_version` 1.0.0, the site should have the [Redis](https://www.drupal.org/project/redis) module present and installed, but it won't yet send requests to the Redis cache server. There's a variable you can set, `drupal_redis_enabled`, which tells Drupal to use Redis for caching (instead of the default Database backend). You can set it in `playbooks/drupal/vars.yml`, or via command line, e.g.:

    $ ansible-playbook -i inventory playbooks/drupal/main.yml --extra-vars "drupal_version=1.0.0 drupal_redis_enabled=true"

### Testing the performance of the Dramble

There are a few tests I like to run to get a general feel for the speed of the entire cluster. When tweaking configuration, adding content, changing modules, etc., it's good to test the overall performance of the cluster using tools like `ab` or `wrk`.

Here are three of the tests I do most often:

  1. Send 100 requests through 10 active connections to the home page, bypassing Nginx's cache (so the requests are routed to the backend webservers, which in turn connect to Redis and MySQL to get data). My Dramble currently gets around 10 requests/second.
      $ ab -n 100 -c 10 http://pidramble.com/?nocache=true
      $ wrk -t1 -c100 -d10 http://pidramble.com/?nocache=true

  2. Send 100 requests through 10 active connections to the home page, bypassing Nginx's cache and also testing Drupal's ability to serve authenticated visitor traffic (`SESSKEY` is an active cookie session key ID, and `VALUE` is that session ID's value). My Dramble currently gets around 8.8 requests/second.
      $ ab -n 100 -c 10 -C "SESSKEY=VALUE" http://pidramble.com/

  3. Send 10,000 requests through 100 active connections to the home page, testing the balancer's ability to send as many requests per second as possible. My Dramble currently gets around 1,800 requests/second with the onboard LAN, or up to 3,200 requests/second [using a Gigabit USB 3.0 LAN adapter](http://www.midwesternmac.com/blogs/jeff-geerling/getting-gigabit-networking).
      $ ab -n 10000 -c 200 http://pidramble.com/
      $ wrk -t4 -c100 -d30 http://pidramble.com/

## Author

This project was started in 2015 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).

Raspberry Pi image used in architecture diagram by [Multicherry](http://commons.wikimedia.org/wiki/User:Multicherry), downloaded from [Wikipedia](http://en.m.wikipedia.org/wiki/File:Raspberry_Pi_2_Model_B_v1.1_top_new_(bg_cut_out).jpg). All other logos are copyright their respective owners.
