I am using my Kali Linux VM to install Docker. I followed along with the Debian version of the instructions to minimize any issues.

The first command that must be used is:
```console
 for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

This is to remove any unofficial docker packages that might have been pre-installed into Kali that would have a conflict with the official Docker.
As you can see in this screenshot, there were no conflicts, so all is good:

![](<Pasted image 20241104192241.png>)

Next, I run the following commands to add Docker's GPG key:
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
![](<Pasted image 20241104192346.png>)

Then I added the repository:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  bookworm stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

This is where the problems began for me. Upon running the final command, `sudo apt-get update`, apt was not very happy with me...

![](<Pasted image 20241104192640.png>)


Because of this, I ended up just using the following command:
`sudo apt-get install -y docker.io`

This seemed to work a lot better, as once I used `sudo docker run hello-world`, it showed that the installation was working properly.

![](<Pasted image 20241115195517.png>)

`sudo apt install docker-compose`

![](<Pasted image 20241115200317.png>)

Everything looks to be relatively in order now, so I moved on to the next step: installing OpenVAS/Greenbone.

`sudo docker pull mikesplain/openvas:latest`

![](<Pasted image 20241115212744.png>)

`sudo docker run -d -p 443:443 --name openvas mikesplain/openvas`

![](<Pasted image 20241117190842.png>)

Now that I can see that the container is up and running, I am able to open a web browser and navigate to https://localhost in order to get to the following page:

![](<Pasted image 20241117191205.png>)

Now, I wait for the logs to indicate that the OpenVAS installation is OK before logging in.

`sudo docker exec -it openvas bash`
```
export FEED=feed.community.greenbone.net
export COMMUNITY_NVT_RSYNC_FEED=rsync://$FEED:/nvt-feed
export COMMUNITY_CERT_RSYNC_FEED=rsync://$FEED:/cert-data
export COMMUNITY_SCAP_RSYNC_FEED=rsync://$FEED:/scap-data

greenbone-nvt-sync
openvasmd --rebuild --progress
greenbone-certdata-sync
greenbone-scapdata-sync
openvasmd --update --verbose --progress

/etc/init.d/openvas-manager restart
/etc/init.d/openvas-scanner restart
```

After an hour of my computer freezing and not getting through all of the prior commands...I gave up and tried to install WordPress via Docker instead.

To get through this process, I utilized https://hostman.com/tutorials/how-to-install-wordpress-using-docker/. 

`sudo nano docker-compose.yml`

Once inside of nano, I pasted the following and saved the file:
```
version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: exampledb
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
    volumes:
      - wordpress_data:/var/www/html
    networks:
      - wordpress_net

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - wordpress_net

volumes:
  wordpress_data:
  db_data:

networks:
  wordpress_net:
    driver: bridge

```
`sudo docker-compose up -d`

Then, I navigated to https://localhost:8080/ to set up WordPress.

![](<Pasted image 20241117213401.png>)

![](<Pasted image 20241117213534.png>)

![](<Pasted image 20241117213602.png>)

Alas, it is done.