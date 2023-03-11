# Setup Home assistant + Mosquitto + Zigbee2mqtt on Orange pi 3 LTS
# project opi3lts_hass
docker-compose.yml: Configuration file for Orangepi 3 LTS + Home Assistant + Mosquitto + Zigbee2mqtt

## Hareware

* Orangepi 3 LTS, 2GB + 8GB onboard storage.
* 64 GB micro SD card (The onboard storage is not suffcient for docker images)
* Zigbee dongle, here using Sonoff_Zigbee_3.0_USB_Dongle_Plus

## Prepare

* Setup Orange pi 3, burn ubuntu server image on the board. or burn it on microsd card. I am running ubuntu and docker on the onboard storage. My docker images are running on the microsd card.
* Setup docker with Orangepi offical manual www.orangepi.org. One extra step for change docker default folder to your microsd card by add daemon.json to /etc/docker, then restart docker service.
* [Optional, but highly recommand for dummies like me who never used docker] setup portainer.io, google recommand reads https://pimylifeup.com/raspberry-pi-portainer/
* [Optional] another tutorial about setup https://medium.com/geekculture/home-assistant-docker-zigbee2mqtt-3d8e0ba02d10 . basiclly same steps, I add extra notes for each step.

## Configuration, let's start!

### step 1, make all folders for docker images
*  SSH Login with root account
* create following folders on the sd card.
  if you have a project folder on the sd card root:

  - sdcard/project/zigbee2mqtt_config/data
  - sdcard/project/mosquitto_config/config
  - sdcard/project/mosquitto_config/data
  - sdcard/project/mosquitto_config/logs
  - sdcard/project/hass_config
  - sdcard/project/docker-compose.yml [tips, download my github copy and make changes with your settings, you should have basic understading of yml format.]
  
### step 2, pull all docker images.

* in a terimail (in my case, in the folder sdcard/project) run `sudo docker compose up -d ` with this command, all docker images will be downloaded. but all the images are need be to setup

### step 3, setup mosquitto broker docker image
* I followed this video https://www.youtube.com/watch?v=L26JY2NH-Ys
  - setup mosquitto broker and users. copy mosquitto configuration file https://raw.githubusercontent.com/eclipse/mosquitto/master/mosquitto.conf to your folder which in the step1, you can use wget command.
  - add a user to mosquitto, 
    - login to docker image `sudo	docker exec -it YOURMOSQUITTODOCKERIMAGENAME sh`, if you install portainer.io, you can just via web interface to login.
    - in the new termial: `mosquitto_passwd ./mosquitto/config/pwfile USERNAME`. if everything goes right, you should see one extra pwfile in your mosquitto config folder.
    - let your mosquitto system know where the userfile is, setup pwfile location via `sudo nano /mosquitto/config/configuration.yaml` ,line 550: `password_file /mosquitto/config/pwfile`
    - save everything, restart the mosquitto docker image.
    
### step 4, setup a (rootless) zigbee2mqtt

* get a copy from offical github. `wget https://raw.githubusercontent.com/Koenkk/zigbee2mqtt/master/data/configuration.yaml -P data`. the configuration is at sdcard/project/zigbee2mqtt_config/data
* Open the configuarion file with favorite text editor. add mosquitto user/passwith which you created in step 3. enable web UI by add ` fronend: true`.
something like this

```
# Home Assistant integration (MQTT discovery)
homeassistant: true

# allow new devices to join
permit_join: true

frontend: true

# MQTT settings
mqtt:
  # MQTT base topic for zigbee2mqtt MQTT messages
  base_topic: zigbee2mqtt
  # MQTT server URL
  server: 'mqtt://localhost'
  # MQTT server authentication, uncomment if required:
  user: 'MOS'
  password: 'passwd'

# Serial settings
serial:
  # Location of CC2531 USB sniffer
  port: /dev/ttyACM0

```
* restart zigbee2mqtt docker image.

### final step
* login hass via http://192.168.X.X:8123, setup your admin account
* add new intergration by searching MQTT for mosquitto. fill in with your setup info. such as, add server: localhost
* at this step, if you dont see any error message in docker images (via portainer.io), you should be able to add zigbee devices.
* open http://192.168.x.x:8080, click permit join all, the paring your zigbee devices.





