# MQTT Simulator

Easy-to-configure MQTT simulator written in [Python 3](https://www.python.org/) to simulate the sending of JSON objects from sensors or devices to a broker.

[Features](#features) •
[Getting Started](#getting-started) •
[Configuration](#configuration) •
[Authors](#authors)

![Simulator Running](images/simulator-running.gif)

## CHarris - Testing

### Topics are *extremely sensitive*

if settings.json - `"devices/myEdgeDevice/messages/events/freezer"`
then mosquitto.conf - `topic devices/myEdgeDevice/messages/events/freezer out 1`

### Resources

- [Mosquitto MQTT broker to IoT Hub/IoT Edge](http://busbyland.com/mosquitto-mqtt-broker-to-iot-hub-iot-edge/)
- https://mosquitto.org/man/mosquitto-conf-5.html
- [Quickstart creates an Azure IoT Edge device on Linux | Microsoft Learn](https://learn.microsoft.com/azure/iot-edge/quickstart-linux?view=iotedge-1.4&viewFallbackFrom=iotedge-2020-11)
- [Quickstart - Send telemetry to Azure IoT Hub (CLI) quickstart | Microsoft Learn](https://learn.microsoft.com/azure/iot-hub/quickstart-send-telemetry-cli)

- [IoT Explorer](https://github.com/Azure/azure-iot-explorer/releases)

- get a sas token - expires every 60 minutes
`((az iot hub generate-sas-token -g charris-iot1 -d chicagoFreezer2 --duration $(60*20*24*365) -n charris-iot1 -o json) | convertfrom-json).sas | set-clipboard`

- test with mosquitto_pub
mosquitto_pub -t "devices/myEdgeDevice/messages/events/freezer" -i "myEdgeDevice" -u "charris-iot1.azure-devices.net/myEdgeDevice/?api-version=2020-09-30" -P "SharedAccessSignature sr=charris-iot1.azure-devices.net%2Fdevices%2FmyEdgeDevice&sig=tj9UrQvd9Y5gVheFoS8m9Peg4QhBG0%2BtjvRthS%2FqVaI%3D&se=1679593447" -h "charris-iot1.azure-devices.net" -V mqttv311 -p 8883 --cafile Baltimore.pem -m 'My Awesome Message' -d

- monitor what's coming into IoT hub
    `az iot hub monitor-events --output table --device-id devicename --hub-name hubname --output json`

### process

1. from the /sensor-monitor folder
2. start the broker
    `docker build -t js/mqtt-broker .\mqtt-broker\.; .\mqtt-broker\Dockerrun.ps1`
3. start the simulator
    `docker build -t js/mqtt-simulator .\mqtt-simulator\.; .\mqtt-simulator\Dockerrun.ps1`
4. install mosquitto client
   `apt install mosquitto -y`
5. subscribe
    `mosquitto_sub -h localhost -p 1883 -u sensor123 -P sensor123 -v -t freezer/+`
    or http://mqtt-explorer.com/ - connect and then click the little green graph icon next to the value on the right side

## Azure Setup

1. https://techcommunity.microsoft.com/t5/internet-of-things-blog/mosquitto-client-tools-and-azure-iot-hub-the-beginning-of/ba-p/2824717
2. 
3. Ignore following...
4. `$LOCATION = "eastus"`
5. `$RESOURCE_GROUP_NAME = "charris-iot1"`
6. `$HUB_NAME = $RESOURCE_GROUP_NAME`
7. `az group create --location $location --name $RESOURCE_GROUP_NAME`
8. `az iot hub device-identity create --device-id myEdgeDevice --edge-enabled --hub-name $HUB_NAME`
9. View connection string
   1. `az iot hub device-identity connection-string show --device-id myEdgeDevice --hub-name $HUB_NAME`
10. Elevated PowerShell
    1. Install AzureIoTEdge

    ```powershell
    $msiPath = $([io.Path]::Combine($env:TEMP, 'AzureIoTEdge.msi'))
    $ProgressPreference = 'SilentlyContinue'
    Invoke-WebRequest "https://aka.ms/AzEFLOWMSI_1_4_LTS_X64" -OutFile $msiPath
    Start-Process -Wait msiexec -ArgumentList "/i","$([io.Path]::Combine($env:TEMP, 'AzureIoTEdge.msi'))","/qn"
    ```

### Debug mqtt-broker

`apk update`
`apk add nano`

## Features

- Small and easy-to-configure simulator for publishing data to a broker
- Configuration from a single JSON file
- Connection on pre-defined fixed topics
- Connection on multiple topics that have a variable id or items at the end
- Random variation of data generated according to configuration parameters

## Getting Started

### Prerequisites

- [Python 3](https://www.python.org/) (with pip)

#### Installing Dependencies

To install all dependencies with a virtual environment:

```shell
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```

#### Running

The default simulator settings can be changed in the `config/settings.json` file.

```shell
python3 mqtt-simulator/main.py
```

Runs the simulator according to the settings file.  
The terminal will show the simulator event log.

Optionally, you can pass a flag with the path to settings file:

```shell
python3 mqtt-simulator/main.py -f <path/settings.json>
```

## Configuration

- The `config/settings.json` file has three main configuration parameters:

    ```json
    {
        "BROKER_URL": "mqtt.eclipse.org",
        "BROKER_PORT": 1883,
        "TOPICS": [
            ...
        ]
    }
    ```

    | Key | Type | Description | Required |
    | --- | --- | --- | --- |
    | `BROKER_URL` | string | The broker URL where the data will be published | yes |
    | `BROKER_PORT` | number | The port used by the broker | yes |
    | `TOPICS` | array\<Objects> | Specification of topics and how they will be published | yes |

- The key **TOPICS** has a array of objects where each one has the format:

    ```json
    {
        "TYPE": "multiple",
        "PREFIX": "temperature",
        "RANGE_START": 1,
        "RANGE_END": 2,
        "TIME_INTERVAL": 25,
        "RETAIN_PROBABILITY": 0.5,
        "DATA": [
            ...
        ]
    }
    ```

    | Key | Type | Description | Required |
    | --- | --- | --- | --- |
    | `TYPE` | string | It can be `"single"`, `"multiple"` or `"list"` | yes |
    | `PREFIX` | string | Prefix of the topic URL, depending on the `TYPE` it can be concatenated to `/<id>` or `/<item>` | yes |
    | `LIST` | array\<any> | When the `TYPE` is `"list"` the topic prefix will be concatenated with `/<item>` for each item in the array | if `TYPE` is `"list"` |
    | `RANGE_START` | number | When the `TYPE` is `"multiple"` the topic prefix will be concatenated with `/<id>` where `RANGE_START` will be the first number  | if `TYPE` is `"multiple"`  |
    | `RANGE_END` | number | When the `TYPE` is `"multiple"` the topic prefix will be concatenated with `/<id>` where `RANGE_END` will be the last number | if `TYPE` is `"multiple"`  |
    | `TIME_INTERVAL` | number | Time interval in seconds between submissions towards the topic | yes |
    | `RETAIN_PROBABILITY` | number | Number between 0 and 1 for the probability of the previous data being retained and sent again | yes |
    | `DATA` | array\<Objects> | Specification of the data that will form the JSON to be sent in the topic | yes |

- The key **DATA** inside TOPICS has a array of objects where each one has the format:

    ```json
    {
        "NAME": "temperature",
        "TYPE": "float",
        "MIN_VALUE": 30,
        "MAX_VALUE": 40,
        "MAX_STEP": 0.2
    }
    ```

    | Key | Type | Description | Required |
    | --- | --- | --- | --- |
    | `NAME` | string | JSON property name to be sent | yes |
    | `TYPE` | string | It can be `"int"`, `"float"` or `"bool"` | yes |
    | `MIN_VALUE` | number | Minimum value that the property can assume | If `TYPE` is different from `"bool"` |
    | `MAX_VALUE` | number | Maximum value that the property can assume | If `TYPE` is different from `"bool"` |
    | `MAX_STEP` | number | Maximum change that can be applied to the property from a published data to the next | If `TYPE` is different from `"bool"` |

## Authors

[![DamascenoRafael](https://github.com/DamascenoRafael.png?size=70)](https://github.com/DamascenoRafael)
 [![Maasouza](https://github.com/Maasouza.png?size=70)](https://github.com/Maasouza)
