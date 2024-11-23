Upon creating an account, this is the first page that you are greeted with.

![](<Pasted image 20241123135448.png>)

For this project, we want to create a virtual machine. This is the page that you come across when you click "Deploy a virtual machine."
We can leave it as San Francisco for the region:

![](<Pasted image 20241123135751.png>)

For the image, we want to use Ubuntu version 24.04.

![](<Pasted image 20241123135850.png>)

The Droplet size can be configured as displayed in the below image (Basic, Regular, $6/mo):

![](<Pasted image 20241123135934.png>)

Authentication was then established with a password:

![](<Pasted image 20241123140105.png>)

And the droplet is officially created!

![](<Pasted image 20241123140326.png>)

To enter the VM, navigate to the Droplet Console section:

![](<Pasted image 20241123140619.png>)

After you click "Launch Droplet Console," you should have this terminal open up in a new window:

![](<Pasted image 20241123140640.png>)

Now we want to use the following commands to get Wireguard set up:

```
mkdir -p ~/wireguard/
mkdir -p ~/wireguard/config/
nano ~/wireguard/docker-compose.yml
```
Once you are in nano, the file contents should look like this:

![](<Pasted image 20241123140828.png>)

Though, some small modifications need to be made to the file. First, change TZ under environment to the following:

![](<Pasted image 20241123141159.png>)

SERVERURL needs to be altered as well:

![](<Pasted image 20241123141501.png>)

Now, CTRL+O, Enter, CTRL+X to save and exit the nano editor.

Before we can move on with Wireguard, Docker must be installed with the following commands:

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

apt-cache policy docker-ce

sudo apt install docker-ce -y

sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

Now we are able to get back to the WireGuard setup.

```
cd ~/wireguard/
docker-compose up -d
```

![](<Pasted image 20241123142349.png>)


Run this command to get the QR code to set up mobile device connection to the VPN:
```
docker-compose logs -f wireguard
```

![](<Pasted image 20241123142813.png>)

I downloaded the Wireguard app on my mobile device. Here is what it looks like when you open it at first:

![](<Screenshot_20241123_142942_WireGuard.jpg>)

Using the "+" button shown in the screenshot above, I scanned the QR code that was displayed in the Ubuntu terminal.

BEFORE activating the VPN:

![](<Screenshot_20241123_143108_Samsung Internet.jpg>)

AFTER activating the VPN:
![](<Screenshot_20241123_143054_Samsung Internet.jpg>)

Now, it is time to test the VPN on my laptop.

Prior to doing anything, here is my current IP address:

![](<Pasted image 20241123143804.png>)

I installed Wireguard, and this is what the GUI looks like:

![](<Pasted image 20241123144218.png>)

I need to add a tunnel which links to what I set up on the Ubuntu VM.

![](<Pasted image 20241123150430.png>)

![](<Pasted image 20241123150354.png>)

All done.