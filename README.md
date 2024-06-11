# Coexistence Testing

This repository contains instructions on how to set up Silicon Labs software and firmware for coexistence testing, focusing on Zigbee and Bluetooth.

## Zigbee Gateway

The Zigbee Gateway consists of a host and an EFR32 device connected via a serial link. There are two main options for this:

1. NCP: The EFR32 is running an NCP firmware image (which includes the Silicon Labs Zigbee stack) and this communicates with a host application (Z3Gateway) via a serial link (either UART or SPI).
2. RCP: The EFR32 is running an RCP firmware image (usually multiprotocol) and the host is running the Zigbee stack (in the form of zigbeed) and the host application (Z3Gateway). The serial connection between the EFR32 and the host is typically implemented using cpc (which helps with multiplexing for multiprotocol applications).

### Software Configuration

#### NCP
On the NCP, the following components are recommended:

1. "Platform->Radio->Rail Utility, Coexistence" - This needs to be configured with your default coex/pta settings, and also needs to be set up properly for your hardware (i.e. GPIO pin selection and signal polarity).


#### Z3Gateway

For Z3Gateway, the following components are recommended:

1. Throughput Library. This is documented in section 4 of [UG350: Silicon Labs Coexistence Development Kit (SLWSTK-COEXBP)](https://www.silabs.com/documents/public/user-guides/ug350-coexistence-development-kit.pdf#page=37).
2. "Platform->Radio->Zigbee Utility, Coexistence CLI" - This gives you access to the "plugin coexistence" commands for run-time control of coexistence/PTA.
3. In the "Zigbee->Utility->Concentrator Support" component, change the "Concentrator Type" from its default of "Low RAM Concentrator" to "High RAM Concentrator". This helps reduce extraneous Route Request messages when using the Throughput Library.

### Joining a Device to the Gateway

Start Z3Gateway. The gateway application will post some output, and then a `Z3Gateway>` prompt signaling it is ready for interaction.

Form a new Zigbee network:
```
Z3Gateway> network leave
Z3Gateway> plugin network-creator start 1
```
When this succeeds it will post `EMBER_NETWORK_UP 0x0000`.

We also want to execute a command to reduce some of the unnecessary traffic (in this case, periodic MTORR broadcasts) so that we can focus on the traffic we are intentionally sending:

```
Z3Gateway> plugin concentrator stop
```

Next open the network on the gateway to join a device. You need to do this shortly before attempting to join as it will eventually time out and have to be re-executed:
```
Z3Gateway> plugin network-creator-security open-network
```

### PTA Control

Configuration of the PTA is done via the Coexistence CLI component on Z3Gateway. This is documented in [AN1017: Zigbee and Silicon Labs Thread Coexistence with Wi-Fi](https://www.silabs.com/documents/public/application-notes/an1017-coexistence-with-wifi.pdf). The commands are summarized in the following table.

| Command                       | Command Description       |
| --------------                |---------------------------|
| coexistence get-dp-state      | Returns the Directional PRIORITY state along with pulse width (us). |
| coexistence get-dp-state      | Set Directional PRIORITY state.   |
| coexistence get-gpio-input    | Returns GPIO Input override from console. |
| coexistence set-gpio-input    | Sets GPIO Input override from console. |
| coexistence get-phy-state     | Returns PHY select state.   |
| coexistence set-phy-state     | Sets PHY select timeout. |
| coexistence get-pta-options   | Returns ptaOptions. |
| coexistence set-pta-options   | Sets ptaOptions.   |
| coexistence get-pta-state     | Returns PTA enable/disable state. |
| coexistence set-pta-state     | Sets PTA enable/disable state. |
| coexistence get-pwm-state     | Returns PWM state from console with period and duty cycle. |
| coexistence set-pwm-state     | Sets PWM state.   |
| coexistence reset-counters    | Clears coex counters. |
| coexistence get-counters      | Returns the PTA specific debug counter values. |

Here are details and examples of a few specific commands:

#### plugin coexistence set-pwm-state <pwmPeriod_halfms> <pwmPulse_DC> <pwmPriority>
 
Set the PTA PWM state
    pwmPeriod_halfms - INT8U - PWM period (0.5ms resolution)
    pwmPulse_DC - INT8U - PWM duty-cycle (%)
    pwmPriority - INT8U - PWM priority (0=low|1=high)
 
Example, to set PWM period to 39.5ms (78 * 0.5ms), PWM duty cycle to 20%, and low priority when PWM REQUEST is asserted:
```
plugin coexistence set-pwm-state 78 20 0
```
Example, to disable PWM:
```
plugin coexistence set-pwm-state 0 0 0
```

#### plugin coexistence get-pwm-state
 
Get the PTA PWM state
 
Example after setting PWM period to 39.5ms (78 * 0.5ms), PWM duty cycle to 20%, and low priority when PWM REQUEST is asserted:
```
plugin coexistence get-pwm-state
PTA PWM (ENABLED): 78 (PERIOD in 0.5ms), 20 (%DC), 0 (LOW PRIORITY)
```

#### plugin coexistence result-counters
```
plugin coexistence result-counters
COUNTERS
PTA Lo Pri Req: 42
PTA Hi Pri Req: 0
PTA Lo Pri Denied: 0
PTA Hi Pri Denied: 0
PTA Lo Pri Tx Abrt: 0
PTA Hi Pri Tx Abrt: 0
```

### PTA Debug Counters

| Counter Index                 | Meaning |
| ----------------              |---------|
| EMBER_COUNTER_PTA_LO_PRI_REQUESTED | Occurrences of REQUEST asserted with low priority. |
| EMBER_COUNTER_PTA_HI_PRI_REQUESTED | Occurrences of REQUEST asserted with high priority. |
| EMBER_COUNTER_PTA_LO_PRI_DENIED    | Occurrences of GRANT denied with low priority REQUEST. |
| EMBER_COUNTER_PTA_HI_PRI_DENIED    | Occurrences of GRANT denied with high priority REQUEST. |
| EMBER_COUNTER_PTA_LO_PRI_TX_ABORTED | Occurrences of TX aborted by GRANT de-asserted with low priority REQUEST. |
| EMBER_COUNTER_PTA_HI_PRI_TX_ABORTED  | Occurrences of TX aborted by GRANT de-asserted with high priority REQUEST. |

## Zigbee End Node

The easiest way to implement a Zigbee End Node for coexistence testing is to run an example project on the Silicon Labs Wireless Starter Kit / Wireless Pro Kit. 

There are a few different test approaches for the Zigbee end node for coexistence testing. One approach is to use the Throughput Library to push a stream of data. The Throughput Library is documented in section 4 of [UG350: Silicon Labs Coexistence Development Kit (SLWSTK-COEXBP)](https://www.silabs.com/documents/public/user-guides/ug350-coexistence-development-kit.pdf#page=37). The Throughput Library approach is good for covering worst case conditions (like a Zigbee OTA). The other approach is to use data polls sent from the node to its parent, which more closely covers steady state use cases involving battery powered devices (door/window sensors, environmental sensors, etc.) for which increased retries has battery life implications.

### Zigbee End Node for Throughput and Data Poll Testing

1. Create the "Zigbee - SoC Light (Z3Light)" example for your radio board.
2. Install the "Zigbee->Utility->Throughput" component.
3. Install "Zigbee->Stack->Pro Core->Pro Leaf Stack" (Not "zigbee_pro_leaf_stack_mac_test_cmds") and accept prompt to replace Zigbee PRO Stack (Library) with Pro Leaf Stack.
4. Under the "Zigbee->Utility->Zigbee Device Config" component, change the "Primary Network Device Type" to "End Device" and leave "Zigbee 3.0 Security".
5. Add "Zigbee->Utility->End Device Support". If you want to have the node automatically send poll requests, configure the long poll interval value to your desired interval for Poll Requests. If you want to send poll requests manually, set the long poll interval to a high value (like 1000 seconds).
6. To send manual poll requests, we need to add some custom CLI code to app.c. Replace the example app.c with the [src/app.c](src/app.c) from this repo.
7. Build and flash. Don't forget the bootloader!

### Joining the Zigbee Node to the Gateway

Once the network has been formed on the Gateway and it is open for joining [see previous section](#Joining-a-Device-to-the-Gateway), then you can join the network via the CLI of your Zigbee Node (same for Z3Light or the "sleepy" Zigbee Minimal):
```
network leave
plugin network-steering start 0
```

You will see a bunch of text printed to the console, and if successful, eventually you will see:
```
Device Announce: 0xNNNN
```
where NNNN is the short network address of your new device (in hex). Note that you will see this on both your Zigbee device console as well as on Z3Gateway. And then eventually you will see:
```
Join network complete: 0x00
```
indicating the successful completion of the joining process.

### Coexistence Testing using Automatic Poll Requests
If you have configured the node to automatically send frequent poll requests, it will begin long polling shortly after joining. You can use the Simplicity Studio Network analyzer to confirm the periodic poll requests are being sent from the Zigbee Node to the Gateway. See below for what this looks like on a 3 second long poll interval with our device short address 0x0B23. Poll requests aren't encrypted, so you don't need to enter in your network key into the Network Analyzer tool to confirm this functionality.
![Simplicity Studio Network Analyzer trace snippet showing data requests with MAC ACKs](images/data%20requests%20on%20packet%20trace.png)

### Coexistence Testing using Manual Poll Requests
The [src/app.c](src/app.c) from this repo adds a custom cli "sendPollRequest" which sends a manual poll request to the parent. 

To enable the callback to print to the console on poll request completion, use:
```
Z3Light> plugin end-device-support poll-completed-callback 1
```

Manually sending poll requests will look like this:
```
Z3Light>sendPollRequest
emberPollForData, status: 0x00
Poll complete, status: 0x31
```

Note that status 0x31 from the Poll complete is normal, as it is EMBER_MAC_NO_DATA and just indicates that the parent ACK for the poll request indicated that the parent had no data to send.

### Coexistence Testing using the Throughput Library

After joining, you can use the Throughput Library to send data in either direction. For example, on the node side:
```
Z3Light> plugin throughput set-all 0x0000 10 5000 80 1 0 100000 
 
TEST PARAMETERS

Destination nodeID: 0x0000 

Packets to send: 10 

Transmit interval: 5000 ms

Payload size: 35B

Packet size: 80B

Packets in flight: 1

APS Options = 0x0000

Timeout: 100000 ms

Z3Light> plugin throughput start
Clearing counters

Starting Test

Test Complete

THROUGHPUT RESULTS

Total time 45021 ms

Success messages: 10 out of 10

Payload Throughput: 62 bits/s

Phy Throughput: 142 bits/s

Min packet send time: 10 ms

Max packet send time: 60 ms

Avg packet send time: 20 ms

STD packet send time: 20 ms
```

You can see these throughput packets highlighted in pink in the packet trace shown below. You can also see a few other packets being generated, although only the two gray "Route Record" traces are unicast packets in the same direction (Node=0x2A7F to Gateway=0x0000).

![packet trace showing throughput packets](images/throughput%20library%20trace.png)

### Using Diagnostic Counters to Measure Coexistence Performance

Silicon Labs Zigbee firmware projects include the diagnostic counters by default. The counters can be read without clearing with:
```
plugin counters simple-print
0) Mac Rx Bcast: 0

1) Mac Tx Bcast: 0

2) Mac Rx Ucast: 0

3) Mac Tx Ucast: 332

4) Mac Tx Ucast Retry: 0

5) Mac Tx Ucast Fail: 2
```
These counters can be read on both Z3Gateway and also via serial console of the Zigbee Node. The most important Zigbee counters for coexistence are the Mac Tx Ucast (3) and Mac Tx Ucast Retry (4) on the Zigbee Node. Mac Tx Ucast Retry / Mac Tx Ucast is the unicast retry rate on the Zigbee node, and this assesses how well the Gateway is receiving the incoming packets from the Zigbee Node, as well as how well the Zigbee Node is able to hear the Gateway's ACK.

On the Gateway, counter #32 (CCA Failures) is also a critical counter for coexistence as CCA failure due to 2.4 GHz energy from the co-located Wi-Fi can cause Ucast failures (counter #5). 

## Bluetooth

TBD...