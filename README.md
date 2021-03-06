# HASS-Deepstack-face
[Home Assistant](https://www.home-assistant.io/) custom components for using Deepstack face detection and recognition. [Deepstack](https://www.deepquestai.com/insider/) is a service which runs in a docker container and exposes deep-learning models via a REST API. There is no cost for using Deepstack, although you will need a machine with 8 GB RAM. On your machine with docker, pull the latest image (approx. 2GB):

```
sudo docker pull deepquestai/deepstack
```
**Recommended OS** Deepstack docker containers are optimised for Linux or Windows 10 Pro. Mac and regular windows users my experience performance issues.

**GPU users** Note that if your machine has an Nvidia GPU you can get a 5 x 20 times performance boost by using the GPU, [read the docs here](https://deepstackpython.readthedocs.io/en/latest/gpuinstall.html#gpuinstall).

**Legacy machine users** If you are using a machine that doesn't support avx or you are having issues with making requests, Deepstack has a specific build for these systems. Use `deepquestai/deepstack:noavx` instead of `deepquestai/deepstack` when you are installing or running Deepstack.

## Activating the API
Before you get started, you will need to activate the Deepstack API. First, go to www.deepstack.cc and sign up for an account. Choose the basic plan which will give us unlimited access for one installation. You will then see an activation key in your portal.

On your machine with docker, run Deepstack without any recognition so you can activate the API on port `5000`:
```
sudo docker run -v localstorage:/datastore -p 5000:5000 deepquestai/deepstack
```

Now go to http://YOUR_SERVER_IP_ADDRESS:5000/ on another computer or the same one running Deepstack. Input your activation key from your portal into the text box below "Enter New Activation Key" and press enter. Now stop your docker container. You are now ready to start using Deepstack!

## Home Assistant setup
Place the `custom_components` folder in your configuration directory (or add its contents to an existing `custom_components` folder). Then configure face recognition . Note that at we use `scan_interval` to (optionally) limit computation, [as described here](https://www.home-assistant.io/components/image_processing/#scan_interval-and-optimising-resources).

## Face recognition
Deepstack [face recognition](https://deepstackpython.readthedocs.io/en/latest/facerecognition.html) counts faces (detection) and (optionally) will recognise them if you have trained your Deepstack using the `deepstack_teach_face` service. In `detect_only` mode processing is faster than recognition mode, but any trained faces will not be listed in the `matched_faces` attribute.

On you machine with docker, run Deepstack with the face recognition service active on port `5000`:
```
sudo docker run -e VISION-FACE=True -e API-KEY="Mysecretkey" -v localstorage:/datastore -p 5000:5000 deepquestai/deepstack
```

The `deepstack_face` component adds an `image_processing` entity where the state of the entity is the total number of faces that are found in the camera image. Recognised faces are listed in the entity `matched faces
` attribute.

Add to your Home-Assistant config:
```yaml
image_processing:
  - platform: deepstack_face
    ip_address: localhost
    port: 5000
    api_key: Mysecretkey
    timeout: 5
    detect_only: True
    scan_interval: 20
    source:
      - entity_id: camera.local_file
        name: face_counter
```
Configuration variables:
- **ip_address**: the ip address of your deepstack instance.
- **port**: the port of your deepstack instance.
- **api_key**: (Optional) Any API key you have set.
- **timeout**: (Optional, default 10 seconds) The timout for requests to deepstack.
- **detect_only**: (Optional, boolean, default `False`) If `True`, only detection is performed. If `False` then recognition is performed.
- **source**: Must be a camera.
- **name**: (Optional) A custom name for the the entity.

#### Service `deepstack_teach_face`
This service is for teaching (or [registering](https://deepstackpython.readthedocs.io/en/latest/facerecognition.html#face-registeration)) faces with deepstack, so that they can be recognised.

Example valid service data:
```
{
  "name": "Adele",
  "file_path": "/config/www/adele.jpeg"
}
```

<p align="center">
<img src="https://github.com/robmarkcole/HASS-Deepstack-face/blob/master/docs/face_usage.png" width="500">
</p>

<p align="center">
<img src="https://github.com/robmarkcole/HASS-Deepstack-face/blob/master/docs/face_detail.png" width="350">
</p>

## Object recognition
For object (e.g. person) recognition with Deepstack use https://github.com/robmarkcole/HASS-Deepstack-object

### Support
For code related issues such as suspected bugs, please open an issue on this repo. For general chat or to discuss Home Assistant specific issues related to configuration or use cases, please [use this thread on the Home Assistant forums](https://community.home-assistant.io/t/face-and-person-detection-with-deepstack-local-and-free/92041).

### Docker tips
Add the `-d` flag to run the container in background, thanks [@arsaboo](https://github.com/arsaboo).

### FAQ
Q1: I get the following warning, is this normal?
```
2019-01-15 06:37:52 WARNING (MainThread) [homeassistant.loader] You are using a custom component for image_processing.deepstack_face which has not been tested by Home Assistant. This component might cause stability problems, be sure to disable it if you do experience issues with Home Assistant.
```
A1: Yes this is normal

------

Q2: Will Deepstack always be free, if so how do these guys make a living?

A2: I'm informed there will always be a basic free version with preloaded models, while there will be an enterprise version with advanced features such as custom models and endpoints, which will be subscription based.

------

Q3: What are the minimum hardware requirements for running Deepstack?

A3. Based on my experience, I would allow 0.5 GB RAM per model.

------

Q4: If I teach (register) a face do I need to re-teach if I restart the container?

A4: So long as you have run the container including `-v localstorage:/datastore` then you do not need to re-teach, as data is persisted between restarts.

------

Q5: I am getting an error from Home Assistant: `Platform error: image_processing - Integration deepstack_object not found`

A5: This can happen when you are running in Docker/Hassio, and indicates that one of the dependencies isn't installed. It is necessary to reboot your Hassio device, or rebuild your Docker container. Note that just restarting Home Assistant will not resolve this.

------