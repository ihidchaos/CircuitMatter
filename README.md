# CircuitMatter

CircuitMatter is a Python-only implementation of the Matter IOT specification. It is aimed at hobby use and hasn't been certified for commercial use.

The Matter spec originates out of the Connected Home over IP (CHIP) project and some resources still use this naming. Matter is the trademark associated with certification.

## Get the Matter Specification
The Matter specification is behind a contact info wall here: https://csa-iot.org/developer-resource/specifications-download-request/ CircuitMatter code is based on version 1.3 and references sections from that version.

You do not need to pay anything or be a member organization.

## Running CircuitMatter

CircuitMatter is currently developed in CPython 3.12, the de facto implementation written in C. It is designed with minimal dependencies so that it can also be used on CircuitPython on microcontrollers.

After cloning the repo, pip install `ecdsa`, `cryptography` and `qrcode`.

### Running a CircuitMatter replay

CircuitMatter can capture and replay UDP packets and random numbers to ease development. You can test the start of the CircuitMatter process by using the replay file from the repo:

```shell
python -m circuitmatter test_data/recorded_packets.jsonl
```

### Running for real

To run CircuitMatter against a live Matter commissioner run:

```shell
python -m circuitmatter
```

This will start up MDNS via avahi for discovery by the commissioner and then reply to received UDP packets. CircuitMatter currently doesn't fully commission so it can't act as any specific type of device yet. When it can, there will be examples.

## Running a Matter commissioner

### chip-tool

The de facto standard implementation of Matter is open source as well. It is written in C++ and has many dependencies. It implements all of the different facets of the specification.

We use this implementation via [ESP Matter](https://github.com/espressif/esp-matter) (tested on commit 9350d9d5f948d3b7c61c8659c4d6990d0ff00ea4) to run an introspectable (aka debug printable) commissioner.

To setup esp-matter clone the repo and load submodules:

```shell
git clone -o espressif git@github.com:espressif/esp-matter.git
cd esp-matter
git submodule update --init --recursive .
```

This will pull down the ESP Matter wrapper code and the projectchip implementation into the `connectedhomeip/connectedhomeip/` sub-directory.

To build all of the command line tools run

```shell
bash install.sh
```

(Or source it directly if you use bash.)

Now setup the environment using `export.sh`. (This depends on what shell you use.)

Next, run `chip-tool` to initiate the commissioning process:

```shell
chip-tool pairing onnetwork 1 67202583
```

This will look up commissionable devices on the network via MDNS and then start that process. `67202583` is the manual pairing code that matches the device state in `test_data/device_state.json`.

Logs can be added into the chip sources to understand what is happening on the commissioner side. To rebuild, I've had to run `bash install.sh` again.

### Apple Home

The Apple Home app can also discover and (attempt to) commission the device. Tap Add Accessory.
* By default this will pull up the camera to scan a QR Code. CircuitMatter will print the qrcode to the console to scan.
* You can also use the passcode by clicking "More options" and the CircuitMatter device will show up as a nearby Matter Accessory. Tap it and then enter the setup code `67202583`. This will start the commissioning process from Apple Home.

## Generate a certificate declaration

Each Matter device has its own certificate to capture whether a device has been certified. (CircuitMatter declares itself as an uncertified test/development/hobby device.) This is referred to the Certificate Declaration (CD). CircuitMatter can generate a CD for you and the remaining certificates are loaded from the projectchip repositorry. The certificate generated by CircuitMatter is signed by a test private key.

To generate a CD run:

```shell
python -m circuitmatter.certificates
```

This will write the `certificate_declaration.der` file. (`test_data/certificate_declaration.der` is checked into the repo too and used by default.)
