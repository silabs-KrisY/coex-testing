# Coexistence Testing

This repository contains instructions on how to set up Silicon Labs software and firmware for coexistence testing, focusing on Zigbee and Bluetooth.

## Zigbee Gateway

The Zigbee Gateway consists of a host and an EFR32 device connected via a serial link. There are two main options for this:

1. NCP: The EFR32 is running an NCP firmware image (which includes the Silicon Labs Zigbee stack) and this communicates with a host application (Z3GatewayHost) via a serial link (either UART or SPI).
2. RCP: The EFR32 is running an RCP firmware image (usually multiprotocol) and the host is running the Zigbee stack (in the form of zigbeed) and the host application (Z3GatewayHost). The serial connection between the EFR32 and the host is typically implemented using cpc (which helps with multiplexing for multiprotocol applications).

### Software Configuration

#### NCP
On the NCP, the following components are recommended:
1. 


#### Z3GatewayHost

### Software Configuration
For Z3GatewayHost, the following components are recommended:
1. 

### Joining a Device

Start Z3Gateway. The gateway application will post some output, and then a `Z3GatewayHost>` prompt signaling it is ready for interaction.

Form a new Zigbee network:
```
Z3GatewayHost> network leave
Z3GatewayHost> plugin network-creator start 1
```
When this succeeds it will post `EMBER_NETWORK_UP 0x0000`.

Open the network on the gateway to join a device:
```
Z3GatewayHost> plugin network-creator-security open-network
```

### PTA Control

Configuration of the PTA is done via the Coexistence CLI component on Z3GatewayHost. This is documented in [AN1017: Zigbee and Silicon Labs Thread Coexistence with Wi-Fi](https://www.silabs.com/documents/public/application-notes/an1017-coexistence-with-wifi.pdf). The commands are summarized in the following table.

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
| coexistence get-pta-state     | Returns PTA state. |
| coexistence set-pta-state     | Sets PTA state. |
| coexistence get-pwm-state     | Returns PWM state from console with period and duty cycle. |
| coexistence set-pwm-state     | Sets PWM state.   |
| coexistence reset-counters    | Clears coex counters. |
| coexistence get-counters      | Returns the PTA specific debug counter values. |

Examples: 


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



