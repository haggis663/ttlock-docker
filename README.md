# ttlock-docker

A restful API for TT lock; handles access tokens and dates

Requires gateway and uses TT lock API 

1) Create an application here: <a href="https://open.ttlock.com/manager">https://open.ttlock.com/manager</a>
<br>2) Register a user in the application - returns a prefixed user that you can add to your lock

```
curl --location -g --request POST 'https://euapi.ttlock.com/v3/user/register' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'clientId=your id' \
--data-urlencode 'clientSecret=your secret' \
--data-urlencode 'username=lockuser' \
--data-urlencode 'password=your application users password md5 hashed' \
--data-urlencode 'date=1650909361599'
```

<a href="https://currentmillis.com">https://currentmillis.com</a> - get the current date
<br>echo "password" | md5 - md5 hash password

3) Add your new prefixed user to your lock in the TT lock app
<br>4) Download docker image: <a href="https://hub.docker.com/r/stevendodd/ttlock">https://hub.docker.com/r/stevendodd/ttlock</a>
<br>5) Create a Docker container

```
docker run \
-e CLIENTID='your id' \
-e CLIENTSECRET='your secret' \
-e LOCKID='your lock id' \
-e USER='your prefixed application user' \
-e PASSWORD='your application users password md5 hashed' \
-p 5000:5000 stevendodd/ttlock
```

6) Add the following sensor and rest command to configuration.yaml in home assistant

<img src="https://community-assets.home-assistant.io/original/4X/8/5/b/85b1d906b54557dd772ced7533c7140b49738bb3.png">

```
rest:
  - scan_interval: 60
    resource: http://192.168.176.3:5000/1234567
    sensor:
      - name: "Back Door TTLock"
        value_template: "OK"
        unique_id: backdoor_ttlock
        json_attributes:
          - "lockFlagPos"
          - "autoLockTime"
          - "electricQuantity"
          - "firmwareRevision"
          - "hardwareRevision"
          - "lockAlias"
          - "modelNum"
          - "passageMode"
          - "passageModeAutoUnlock"
          - "soundVolume"
          - "tamperAlert"
  - scan_interval: 60
    resource: http://192.168.176.3:5000/7654321
    sensor:
      - name: "Inner Door TTLock"
        value_template: "OK"
        unique_id: inner_door_ttlock
        json_attributes:
          - "lockFlagPos"
          - "autoLockTime"
          - "electricQuantity"
          - "firmwareRevision"
          - "hardwareRevision"
          - "lockAlias"
          - "modelNum"
          - "passageMode"
          - "passageModeAutoUnlock"
          - "soundVolume"
          - "tamperAlert"
  - scan_interval: 60
    resource: http://192.168.176.3:5000/1234567/getstatus
    sensor:
      - name: "Back Door Status"
        value_template: "{{ value_json.state }}"
        unique_id: backdoor_ttlock_status
        json_attributes:
          - "sensorState"
          - "state"
  - scan_interval: 60
    resource: http://192.168.176.3:5000/7654321/getstatus
    sensor:
      - name: "Inner Door Status"
        value_template: "{{ value_json.state }}"
        unique_id: innerdoor_ttlock_status
        json_attributes:
          - "sensorState"
          - "state"
rest_command:
  unlock_backdoor:
    url: "http://192.168.176.3:5000/1234567/unlock"
    method: get
  unlock_innerdoor:
    url: "http://192.168.176.3:5000/7654321/unlock"
    method: get
  lock_backdoor:
    url: "http://192.168.176.3:5000/1234567/lock"
    method: get
  lock_innerdoor:
    url: "http://192.168.176.3:5000/7654321/lock"
    method: get
lock: 
    - platform: template
      name: Storeroom Inner Door
      value_template: "{{ state_attr('sensor.inner_door_status', 'state') | int != 1 }}" #"{{ is_state('sensor.inner_door_status', '0') }}" 
      lock: 
      - service: rest_command.lock_innerdoor 
      unlock: 
      - service: rest_command.unlock_innerdoor
    - platform: template
      name: Storeroom Outer Door
      value_template: "{{ state_attr('sensor.back_door_status', 'state') | int != 1 }}" 
      lock: 
      - service: rest_command.lock_backdoor 
      unlock: 
      - service: rest_command.unlock_backdoor 
sensor:
  - platform: template
    sensors:
      ttlock_innerdoor_battery:
        friendly_name: "TTLock Inner Door Battery"
        value_template:  '{{ states.sensor.inner_door_ttlock.attributes.electricQuantity|float }}'
        unit_of_measurement: '%'
        device_class: battery
      ttlock_backdoor_battery:
        friendly_name: "TTLock Back Door Battery"
        value_template:  '{{ states.sensor.back_door_ttlock.attributes.electricQuantity|float }}'
        unit_of_measurement: '%'
        device_class: battery

```
