---
layout: page
title: "Pilight"
description: "Instructions how to setup Pilight within Home Assistant."
date: 2015-08-07 18:00
sidebar: true
comments: false
sharing: true
footer: true
logo: pilight.png
ha_category: Hub
ha_release: 0.26
ha_iot_class: "Local Push"
---

[Pilight](https://www.pilight.org/) is a modular and open source solution to communicate with 433 MHz devices and runs on various small form factor computers. A lot of common [protocols](https://wiki.pilight.org/doku.php/protocols) are already available.

This pilight hub connects to the [pilight-daemon](https://wiki.pilight.org/doku.php/pdaemon) via a socket connection to receive and send codes. Thus home assistant does not have to run on the computer in charge of the RF communication. 

The received and supported RF codes are put on the event bus of home assistant and are therefore directly usable by other components (e.g. automation). Additionally a send service is provided to send RF codes.

To integrate pilight into Home Assistant, add the following section to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
pilight:
  host: 127.0.0.1
  port: 5000
```

Configuration variables:

- **host** (*Required*): The IP address of the computer running the pilight-daemon, e.g. 192.168.1.32.
- **port** (*Required*): The network port to connect to. The usual port is [5000](https://www.pilight.org/development/api/).
- **whitelist** (*Optional*): You can define a whitelist to prevent that too many unwanted RF codes (e.g. the neighbours weather station) are put on your HA event bus. All defined subsections have to be matched. A subsection is matched if one of the items are true.

In this example only received RF codes using a daycom or intertechno protocol are put on the event bus and only when the device id is 42. For more possible settings please look at the receiver section of the pilight [API](https://www.pilight.org/development/api/).

A full configuration sample could look like the sample below:

```yaml
# Example configuration.yaml entry
pilight:
  host: 127.0.0.1
  port: 5000
  whitelist:  # optional
    protocol:
      - daycom
      - intertechno
    id:
      - 42
```

## {% linkable_title Troubleshooting %}

- A list of tested RF transceiver hardware is available [here](https://wiki.pilight.org/doku.php/electronics). This might be usefull before buying.
- Sending commands is simple when the protocol is known by pilight, but receiving commands can be rather difficult. It can happend that the code is not correctly recognized due to different timings in the sending hardware or the RF receiver. If this happens follow these steps:

1. [Install](https://www.pilight.org/get-started/installation/) pilight from source (do not worry that is very easy) and only activate the protocols you are expecting in the pop up menu. This reduces false positives.
2. Check the real timings of your device + RF receiver by running `pilight-debug`. Remember the `pulslen` parameter.
3. Go to the `libs/pilight/protocols/433.92` subfolder of the pilight source code and open the .c file of your protocol. Search for `MIN_PULSE_LENGTH`, `MAX_PULSE_LENGTH ` and `AVG_PULSE_LENGTH`. Change the pulse lengths to match your measured one. Recompile and install pilight by re-running `$ sudo ./setup.sh`.
