# INSTALL A PRIVATE REGISTRY SERVER WITHOUT DOMAIN AND SSL CERT
This instructions below will help you set up an registry server with minimum requirement (only thing we need is an Ubuntu server).


## DOCKER REGISTRY SERVER
### First, you need to generate key file
```
mkdir -p certs
openssl req -newkey rsa:4096 -nodes -sha256 -keyout ./certs/domain.key -x509 -days 365 -out ./certs/domain.crt  
```
NOTE: in common name (SERVER FQDN or YOUR NAME): You must set it with domain name ex docker.fuckme.com

### Simple authorization
```
mkdir auth
docker run \
  --entrypoint htpasswd registry:2 -Bbn USERNAME_HERE PASSWORD_HERE > auth/htpasswd
``` 

### Docker Compose Setup
Install Docker Compose by flow this instructions `https://docs.docker.com/compose/install/`

```
nano docker-compose.yml
```
```
version: "3.6"

services:
  registry:
    image: registry:2
    ports:
      - 5000:5000
    environment:
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
      REGISTRY_HTTP_TLS_KEY: /certs/domain.key
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
    volumes:
      - /mnt/volume_sgp1_01/registry_images:/var/lib/registry
      - ./certs:/certs
      - ./auth:/auth
```
Note, you should replace `/mnt/volume_sgp1_01/registry_images` with your image directory 
We will map `certs` and `auth` directory into new container then use it for SSL verifying and authenticating  

### open port 5000 on firewall
```
ufw allow 5000/tcp
ufw reload
ufw enable
```

### Start docker compose

```docker-compose up```

If everything is fine, you can run docker-compose with daemon mode

```docker-compose up -d```

## REGISTRY CLIENT
We need to setup domain and certificate for each registry client.

### SSL Certficate
on `registry server`, copy content of file `~/certs/domain.crt`
```
cat ~/certs/domain.crt
```

then paste it into your client machine
If ubuntu 
```
sudo nano /usr/local/share/ca-certificates/docker_fuckme_com.crt
# paste content of ~/certs/domain.crt here then save
sudo update-ca-certificates
sudo service docker restart

```
If MacOS
You need to create a text file with name `registry_cert.crt" then paste content that you copy from server then open it (with Keychain Access app)
flow the instructions to install it.  
Restart your docker service

### Edit host file
For Ubuntu
```
sudo nano /etc/hosts
# add your registry server ip with your domain here
123.123.123.123 docker.fuckme.com 
```
With OSX, we do the same

### Login
```
docker login docker.fuckme.com:5000 --username YOUR_USERNAME --password YOUR_PASSWORD
```
this is a less secure way, just ignore `--username` and `--password`, you can type it when docker asks

Enjoy

