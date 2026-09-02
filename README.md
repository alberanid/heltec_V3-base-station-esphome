# Heltec WiFi LoRa 32 V3 mailbox base station

ESPHome firmware for a [Heltec WiFi LoRa 32 V3](https://heltec.org/project/wifi-lora-32-v3/) that receives mailbox sensor packets from a CubeCell transmitter and exposes the decoded state to Home Assistant.

The base station listens continuously on 868 MHz, accepts the project's five-byte mailbox packet, and publishes the hatch, door, battery, RSSI, and SNR values through two paths:

- native ESPHome API entities in Home Assistant;
- one JSON message on the MQTT topic `lora/mailbox`.

The configuration also exposes the board's onboard LED as a Home Assistant switch and turns it off at boot.

## Hardware and radio configuration

The firmware targets an ESP32-S3 with 8 MB of flash and the Heltec board's integrated SX1262 radio.

| Function | Pin or setting |
| --- | --- |
| SPI clock | GPIO9 |
| SPI MOSI | GPIO10 |
| SPI MISO | GPIO11 |
| SX1262 chip select | GPIO8 |
| SX1262 DIO1 | GPIO14 |
| SX1262 reset | GPIO12 |
| SX1262 busy | GPIO13 |
| Onboard LED | GPIO35 |
| Frequency | 868 MHz |
| Bandwidth | 125 kHz |
| Spreading factor | SF8 |
| Coding rate | 4/8 |
| Preamble | 8 symbols |
| Sync word | `0x14 0x24` |

The transmitter must use the same frequency, modulation parameters, preamble, sync word, explicit header mode, and CRC setting. Verify that 868 MHz operation and the configured transmit power comply with the radio regulations for the deployment region.

## Packet format

The receiver currently recognizes packets whose first two bytes are `0x01 0x01` and whose total length is at least five bytes.

| Byte | Meaning | Encoding |
| --- | --- | --- |
| 0 | Device ID | `0x01` |
| 1 | Message type | `0x01` |
| 2 | Hatch state | zero is closed; nonzero is open |
| 3 | Door state | zero is closed; nonzero is open |
| 4 | Battery | integer percentage |

Extra bytes are ignored. Accepted packets produce MQTT payloads such as:

```json
{"hatch":false,"door":true,"battery":87,"rssi":-104.5,"snr":7.2}
```

## Home Assistant entities

The native ESPHome API creates these entities:

| Entity name | Type | Unit |
| --- | --- | --- |
| LoRa Mailbox Hatch | binary sensor (`door`) | — |
| LoRa Mailbox Door | binary sensor (`door`) | — |
| LoRa Mailbox Battery | sensor | `%` |
| LoRa Mailbox RSSI | sensor | `dBm` |
| LoRa Mailbox SNR | sensor | `dB` |
| Onboard LED | switch | — |

MQTT telemetry is enabled and uses the `lora/mailbox` topic. The commented `interval` block in the YAML contains an older MQTT discovery approach; it is disabled because Home Assistant can discover the native ESPHome API entities directly.

## Configuration

Create a `secrets.yaml` file beside `heltec_V3-base-station.yaml`:

```yaml
wifi_ssid: "your-wifi-name"
wifi_password: "your-wifi-password"
fallback_wifi_ssid: "LoRa Base Station Fallback"
fallback_wifi_password: "a-strong-fallback-password"
ota_password: "a-strong-ota-password"
api_encryption_key: "a-base64-encoded-32-byte-key"
mqtt_broker_ip: "192.0.2.10"
mqtt_username: "your-mqtt-username"
mqtt_password: "your-mqtt-password"
```

`secrets.yaml` and ESPHome's generated `.esphome/` directory are excluded by `.gitignore`. Do not commit a populated secrets file.

## Build, flash, and monitor

With the board connected over USB and Docker installed, compile and flash the firmware with:

```sh
docker run --privileged --rm \
  -v "${PWD}":/config \
  -it ghcr.io/esphome/esphome \
  run heltec_V3-base-station.yaml
```

To attach to the device logs later, run:

```sh
docker run --privileged --rm \
  -v "${PWD}":/config \
  -it ghcr.io/esphome/esphome \
  logs heltec_V3-base-station.yaml
```

For less verbose operation after commissioning, change `logger.level` from `DEBUG` to `INFO`.

## Known limitation

The packet handler logs unknown or undersized frames, but its current state-update statements are outside the validation branch. A malformed frame can therefore reach those statements with uninitialized decoded values. The current deployment assumes correctly formatted input; for noisy or untrusted RF environments, change the handler to return immediately for invalid packets.

## License

Licensed under the Apache License, Version 2.0. See [LICENSE.txt](LICENSE.txt).
