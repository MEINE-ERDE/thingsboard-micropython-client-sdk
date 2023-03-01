# ThingsBoard MQTT client uPython SDK
[![Join the chat at https://gitter.im/thingsboard/chat](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/thingsboard/chat?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

<a href="https://thingsboard.io"><img src="./logo.png?raw=true" width="100" height="100"></a>

**💡 Make the notion that it is the early alpha of MQTT client uPython SDK special for controllers. So we appreciate any 
help in improving this project and getting it growing.**

ThingsBoard is an open-source IoT platform for data collection, processing, visualization, and device management.
This project is a uPython library that provides convenient client SDK for Device API using MicroPython.

SDK supports:
- Provided all supported feature of umqtt library
- Unencrypted and encrypted (TLS v1.2) connection
- QoS 0 and 1 (MQTT only)
- Automatic reconnect
- [Device MQTT](https://thingsboard.io/docs/reference/mqtt-api/) API provided by ThingsBoard
- Firmware updates
  - Device Claiming are not supported yet 

The [Device MQTT](https://thingsboard.io/docs/reference/mqtt-api/) API are based on uMQTT library.

**For now library support only local install (not from package manager relates to MicroPython)**

## Getting Started

Client initialization and telemetry publishing
### MQTT
```python
from tb_device_mqtt import TBDeviceMqttClient
telemetry = {"temperature": 41.9, "enabled": False, "currentFirmwareVersion": "v1.2.2"}
client = TBDeviceMqttClient("127.0.0.1", "A1_TEST_TOKEN")
# Connect to ThingsBoard
client.connect()
# Sending telemetry without checking the delivery status
client.send_telemetry(telemetry) 
# Sending telemetry and checking the delivery status (QoS = 1 by default)
result = client.send_telemetry(telemetry)
# Disconnect from ThingsBoard
client.disconnect()
```

## Using Device APIs

**TBDeviceMqttClient** provides access to Device MQTT APIs of ThingsBoard platform. It allows to publish telemetry and attribute updates, subscribe to attribute changes, send and receive RPC commands, etc. Use **TBHTTPClient** for the Device HTTP API.
#### Subscription to attributes
##### MQTT
```python
import time
from tb_device_mqtt import TBDeviceMqttClient

def on_attributes_change(client, result, exception):
    if exception is not None:
        print("Exception: " + str(exception))
    else:
        print(result)

client = TBDeviceMqttClient("127.0.0.1", "A1_TEST_TOKEN")
client.connect()
client.subscribe_to_attribute("uploadFrequency", on_attributes_change)
client.subscribe_to_all_attributes(on_attributes_change)
while True:
    client.wait_for_msg()
    time.sleep(1)
```

#### Telemetry pack sending
##### MQTT
```python
import logging
from tb_device_mqtt import TBDeviceMqttClient
import time
telemetry_with_ts = {"ts": int(round(time.time() * 1000)), "values": {"temperature": 42.1, "humidity": 70}}
client = TBDeviceMqttClient("127.0.0.1", "A1_TEST_TOKEN")
client.connect()
results = []
result = True
for i in range(0, 100):
    results.append(client.send_telemetry(telemetry_with_ts))

print("Result " + str(result))
client.disconnect()
```

#### Request attributes from server
##### MQTT
```python
import logging
import time
from tb_device_mqtt import TBDeviceMqttClient

def on_attributes_change(client,result, exception:
    if exception is not None:
        print("Exception: " + str(exception))
    else:
        print(result)

client = TBDeviceMqttClient("127.0.0.1", "A1_TEST_TOKEN")
client.connect()
client.request_attributes(["configuration","targetFirmwareVersion"], callback=on_attributes_change)
while True:
    time.sleep(1)
```

#### Respond to server RPC call
##### MQTT
```python
import psutil
import time
import logging
from tb_device_mqtt import TBDeviceMqttClient

# dependently of request method we send different data back
def on_server_side_rpc_request(client, request_id, request_body):
    print(request_id, request_body)
    if request_body["method"] == "getCPULoad":
        client.send_rpc_reply(request_id, {"CPU percent": psutil.cpu_percent()})
    elif request_body["method"] == "getMemoryUsage":
        client.send_rpc_reply(request_id, {"Memory": psutil.virtual_memory().percent})

client = TBDeviceMqttClient("127.0.0.1", "A1_TEST_TOKEN")
client.set_server_side_rpc_request_handler(on_server_side_rpc_request)
client.connect()
while True:
    time.sleep(1)
```

## Other Examples

There are more examples for both [device](https://github.com/thingsboard/thingsboard-python-client-sdk/tree/master/examples/device) and [gateway](https://github.com/thingsboard/thingsboard-python-client-sdk/tree/master/examples/gateway) in corresponding [folders](https://github.com/thingsboard/thingsboard-python-client-sdk/tree/master/examples).

## Support

 - [Community chat](https://gitter.im/thingsboard/chat)
 - [Q&A forum](https://groups.google.com/forum/#!forum/thingsboard)
 - [Stackoverflow](http://stackoverflow.com/questions/tagged/thingsboard)

## Licenses

This project is released under [Apache 2.0 License](./LICENSE).