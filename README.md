# DockuWiki
A good old fashioned self backup-ing doku**wiki** in a box (container).


### But what is it?
A self installing instance of the venerable [dokuwiki](https://dokuwiki.org) that backs itself up to a git repo every hour. All wrapped up in a docker image (sorry, bow not included).


### But why is it?
We wanted a dead simple method of setting up wikis without hassle. Using a git repo for backups simplifies deployment and eliminates the need for a host with persistent storage.


### I don’t get the name
We tried to be cute by combining “Docker” and “Doku”. Get it? Hah!


### How do I use it?
1. Create a git repo somewhere that the docker host can access. Just don’t commit anything to it! In step 2, let’s pretend you used BitBucket for hosting the repo.


2. ```docker run -d --restart=always --name=wiki -e SSH_DOMAIN=bitbucket.org -e REMOTE_URL=git@bitbucket.org:USERNAME/wiki.git -p 3000:3000 ericbarch/dockuwiki```


3. On the first run, the container will generate a unique SSH key. ```docker logs wiki``` to get the public key of the wiki’s git user that needs to be added to your git server host.


4. Add the SSH key, wait a few moments, and access your freshly minted wiki at http://DOCKERHOST:3000


### What if I accidentally ignite my thermite packed PC and need to redeploy the wiki to a new machine?
Just run the same docker command again and add the new SSH key it generates. If dockuwiki finds an existing wiki, it clones and hosts it.


### Can I get that with SSL?
Nope. TLS only, baby. But that’s where Let’s Encrypt saves the day:

1. Start nginx with the 3 volumes declared:
```bash
$ docker run -d --restart=always --name=nginxproxy \
    -p 80:80 -p 443:443 \
    --name nginx-proxy \
    -v /etc/nginx/certs \
    -v /etc/nginx/vhost.d \
    -v /usr/share/nginx/html \
    -v /var/run/docker.sock:/tmp/docker.sock:ro \
    jwilder/nginx-proxy
```

2. Start the let’s encrypt container:
```bash
$ docker run -d --restart=always --name=letsencrypt \
    --volumes-from nginx-proxy \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    jrcs/letsencrypt-nginx-proxy-companion
```

3. Start the dockuwiki container with your domain name and email:

```bash
$ docker run -d --restart=always --name=wiki \ 
-e SSH_DOMAIN=bitbucket.org -e REMOTE_URL=git@bitbucket.org:USERNAME/wiki.git \
-e "VIRTUAL_HOST=foo.bar.com" -e “LETSENCRYPT_HOST=foo.bar.com“ \
-e “LETSENCRYPT_EMAIL=youremail@yourdomain.com” ericbarch/dockuwiki
```


### How does it work?
Honestly, your guess is as good as mine. And that guess is a few simple bash scripts that fetch the latest stable of dokuwiki and perform an hourly git commit and push.


### Can I run it on a Pi?
Sure, we’re not going to tell you how to live your life:

1. Follow the steps that these awesome guys put together: http://blog.hypriot.com/post/run-docker-rpi3-with-wifi/


2. Then just:
```bash
$ docker run -d --restart=always --name=wiki \
-e SSH_DOMAIN=bitbucket.org -e REMOTE_URL=git@bitbucket.org:USERNAME/wiki.git \
-p 3000:3000 ericbarch/dockuwiki:rpi
```


### Got any config recommendations?
We live and die by [indexmenu](https://www.dokuwiki.org/plugin:indexmenu), [upgrade](https://www.dokuwiki.org/plugin:upgrade), and [bootstrap3](https://www.dokuwiki.org/template:bootstrap3). You can install all of these from the built in extension manager in your dockuwiki instance.


### I’m going on vacation to the moon and won’t have internet access. How can I access my wiki?
No problemo, moon bound traveler. Either take the machine with you that hosts dockuwiki, or use something like a Raspberry Pi to deploy a new instance with the same repo URL. We don’t suggest running multiple instances simultaneously, but your stand in wiki host will collect all your changes and continue attempting to reach the internet until it is safely back on Earth.


### Who created this?
A dude named Eric Barch that is hoping the community will embrace the project so he can collect mad internet karma and retire early.
