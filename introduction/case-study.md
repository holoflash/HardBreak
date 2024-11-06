---
icon: play
---

# Case Study

Scenario: You own a device you want to investigate and maybe modify.

## Non-invasive Testing

As told in the [Methodology ](quickstart/)chapter, first you should try non-invasive methods, as opening the device is risky and can break components or the whole device. Hence, start with:

* Read the documentation:
  * Documentation of IoT devices can reveal a lot of the functionalities
    * For example: Is there a backup function, which writes backups to a SD-card /USB-Stick?
  * Try to find functionalities, which can be exploited
  * Try searching for default passwords, which may give access to more functionalities/data
* Does the device have a webserver running?
  * Try to find common vulnerabilities like RCE, LFI etc.
* Check with Wireshark / RF Analyzer for any communication of the device

## More invasive Testing

If the attempts above are exhausted, we can start with our hardware hacking.

{% hint style="warning" %}
Opening a device comes at the risk of breaking it! Watch out for tamper protection!
{% endhint %}

To do the basic hardware hacking, you just need:

* An [multimeter](../hardware-hacking/basics/tools/hardware-tools/multimeters-and-oscilloscopes.md)
* an [UART to TTL](../hardware-hacking/basics/tools/hardware-tools/uart-to-ttl-adapter.md) USB adapter
* jumper cables
* and in some cases: a [soldering station](../hardware-hacking/basics/tools/hardware-tools/soldering-tools.md)

After opening the device follow:

1. Get an overview of what is available of the PCB board
   1. Checkout which chips are used&#x20;
      1. Google the datasheet of each chip you find (model should be printed on top of the chip)
      2.  It can be useful to take a picture of the PCB and label everything you can identify

          <figure><img src="../.gitbook/assets/komplett-beschriftet (1).png" alt="" width="375"><figcaption><p>Example layout of an PCB</p></figcaption></figure>
   2.  We should also remove shields which prevent us from seeing the hardware:

       <figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption><p>chips under shield</p></figcaption></figure>
   3.  Check for connector or test pads (can be quick wins to find a UART/JTAG etc.)&#x20;

       1. Even better if we find actual pins, where we can connect jumper cables to:

       <figure><img src="../.gitbook/assets/image (31).png" alt="" width="257"><figcaption><p>UART pins exposed</p></figcaption></figure>

       1. JTAG (where we need more pins) are also very interesting targets

       <figure><img src="../.gitbook/assets/image (67).png" alt="" width="82"><figcaption><p>JTAG-Connector</p></figcaption></figure>

       <figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption><p>UART and JTAG pads found</p></figcaption></figure>

       Note: Not all PCBs have these connectors or the interfaces may be disabled.
2.  Check the pinout for the connectors:

    1. Put the multimeter in continuity mode (often a "diode" / "soundwave line**"** symbol) here on top:

    <figure><img src="../.gitbook/assets/image (74).png" alt="" width="507"><figcaption><p>multimeter</p></figcaption></figure>

    1. This mode will check if there is a direct connection between two points on the PCB
    2. Put one probe on the connector pad you want to test
    3.  The other one goes on the chip (datasheet will tell you what pins are used for UART/SPI/JTAG)

        <figure><img src="../.gitbook/assets/image (4).png" alt="" width="287"><figcaption><p>How to probe</p></figcaption></figure>

    You need to find the GND (ground), TX (transmit) and RX (receive) pins to communicate with UART.
3. Another method to figure out the pinout is by looking at the voltage of the pins:
   1. GND should be at 0V
   2. TX pin should fluctuate between 2-3V, depending if there is output or not
   3. RX pin can look like the GND pin, since it just waits for data to come in
4.  Now you need to connect the pins using jumper cables to the UART-USB-TTL adapter (make sure RX -> TX and TX->RX, as they have to be reversed). This can be done by soldering the cables onto the connector pins, plug them in or use clamps.

    <figure><img src="../.gitbook/assets/image (68).png" alt="" width="259"><figcaption><p>UART connection found on PCB</p></figcaption></figure>
5. On your PC use the following command to communicate over UART (you may have to adjust the baud rate)

{% tabs %}
{% tab title="Linux" %}
```bash
sudo minicom -D /dev/ttyUSB0 -b 115200
sudo picocom -b 115200 -r -l /dev/ttyUSB0
```
{% endtab %}

{% tab title="Windows" %}
**Using PuTTY (Windows)**:

* Select “Serial” and enter the COM port (e.g., COM3) and baud rate (115200).
{% endtab %}
{% endtabs %}

5. If you see something like this: You done it correctly!

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption><p>Example bootlog</p></figcaption></figure>

**Congrats!** You found your first serial connection! Check out the UART chapter on how to use this to dump the firmware from the device.

## Resources:

[https://riverloopsecurity.com/blog/2020/01/hw-101-uart/](https://riverloopsecurity.com/blog/2020/01/hw-101-uart/)