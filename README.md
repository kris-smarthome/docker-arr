# 'arr stack
Services:

- prowlarr: indexer proxy
- sonarr: tv media management
- radarr: movie media management
- recyclarr: TRaSH guides 

## usage
Clone this repository to your local machine, edit the config file to match your environment, and run docker compose.

```bash
git clone https://github.com/kris-smarthome/docker-arr.git
cd docker-arr
nano .env
docker compose up -d
```
#### media user
Create the media user and group GUID:10000 GPID:10000


```bash
sudo groupadd -g 10000 media
sudo useradd media -u 10000 -g 10000 -s /bin/bash
```

Set the media user password  `passwd media` using `YOUR_PASSWORD`
Add the default user to the media group `sudo usermod -aG media $USER`