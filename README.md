# Heltec V3 base station

## Build

Run:

```sh
docker run --privileged --rm -v "${PWD}":/config -it ghcr.io/esphome/esphome run heltec_V3-base-station.yaml
```

## Debug

Run:

```sh
docker run --privileged --rm -v "${PWD}":/config -it ghcr.io/esphome/esphome logs heltec_V3-base-station.yaml
```
