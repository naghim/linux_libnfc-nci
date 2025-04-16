# Setup

Háromféleképp lehet kommunikálni ezekkel az NFC chipekkel:

1. PN5xx linux kernel driverrel, amit bele kell égetni a kernelbe
2. I2C driverrel
3. LPC USB Serial IO driverrel (erre külön chip van az NFC lapon, **ezt kell hasznaljuk**)

## Kompilálás

```bash
./bootstrap
./configure --enable-lpcusbsio
make -j16
sudo make install
```

A device allomány: `/dev/hidraw4`: az LPC USB Serial IO innen érhető el, ez a device file

### permission probléma

A userek nem érik el by default... rooté
Két megoldás erre:

- `sudo chmod a+rw /dev/hidraw4`, de ez csak ideiglenes megoldás. ha disconnect van, újra kell futtatni...

- UDEV szabállyal beállítjuk, hogy mindig egy bizonyos féleképp állítsa be a rendszer ezt a fájlt.
  `sudo nano /etc/udev/rules.d/99-nxp-nfc.rules`

Tartalma ez legyen:

```bash
KERNEL=="hidraw*", ATTRS{idVendor}=="1fc9", ATTRS{idProduct}=="0088", MODE="0666"
```

Majd reload command:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

## Demó program

Demó programmal lehet tesztelni:

```bash
./nfcDemoApp poll
```

Ez várja az NFC tageket es kiprinteli.

### üzemmódok:

- `poll` ## Polling mode e.g. <nfcDemoApp poll >
- `write` ## Write tag e.g. <nfcDemoApp write --type=Text -l en -r "Test">
- `share` ## Tag Emulation Mode e.g. <nfcDemoApp share -t URI -u http://www.nxp.com>
- `push` ## Push to device e.g. <nfcDemoApp push -t URI -u http://www.nxp.com>

## Debug

Ha nem műkodik valami:

```bash
sudo nano /usr/local/etc/libnfc-nxp-init.conf
```

az NXPLOG változókat át kell írni 0x00-rol 0x03 log levelre, ez a DEBUG, mindent ki fog irni.

Meg kell nézni ha megvan-e az lsusb command outputban az NFC:

```bash
sudo lsusb
```

> Bus 003 Device 008: ID 1fc9:0088 NXP Semiconductors LPC I2C HID

Meg kell nézni, ha sikerült-e inicializálni az USB-t:

```bash
$ sudo dmesg
7
[ 7310.806883] usb 3-2: new full-speed USB device number 8 using xhci_hcd
[ 7310.958893] usb 3-2: New USB device found, idVendor=1fc9, idProduct=0088, bcdDevice= 1.00
[ 7310.958902] usb 3-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 7310.958905] usb 3-2: Product: LPC I2C HID
[ 7310.958907] usb 3-2: Manufacturer: NXP Semiconductors
[ 7310.958909] usb 3-2: SerialNumber: ABCD123456789
[ 7310.978086] hid-generic 0003:1FC9:0088.000A: hiddev98,hidraw4: USB HID v1.11 Device [NXP Semiconductors LPC I2C HID ] on usb-0000:06:00.4-2/input0
```
