# RPH0002 Battery Arduino Fix

This repository is to help you get your RPH0002 battery working using a arduino and a CAN bus module.

## Items needed: 
- [MCP2515 CAN Bus Module](https://amzn.eu/d/0cbae30o)
- [Small Arduino](https://amzn.eu/d/0eeHnjHA)
- Battery Base with pins
- Battery Base connector cable (Julet 5-Pin)

## Instructions :closed_book:
1. Open Arduino IDE + Arduino and select the correct COM port
2. Go to Sketch > Include Library > Manage Libraries and Search for and install "MCP_CAN" by Cory J Fowler
3. Upload the code below to your arduino
```cpp
#include <mcp_can.h>
#include <SPI.h>

const int SPI_CS_PIN = 10;
const int CAN_INT_PIN = 2;

MCP_CAN CAN(SPI_CS_PIN); // Set CS pin

void setup() {
  Serial.begin(115200);
  
  // Initialize MCP2515 at 250 kbps
  if (CAN.begin(MCP_STDEXT, CAN_250KBPS, MCP_8MHZ) == CAN_OK) {
    Serial.println("CAN BUS Shield init ok!");
  } else {
    Serial.println("CAN BUS Shield init fail");
    while (1);
  }

  pinMode(CAN_INT_PIN, INPUT);
  
  CAN.setMode(MCP_NORMAL); // Start CAN in normal mode
}

void loop() {
  // Message 1
  unsigned char data1[] = {0x04, 0x3B, 0x8F, 0x00, 0x00, 0x2B, 0x04, 0x12};
  // Send message with ID 0x03FF1602 (extended ID)
  CAN.sendMsgBuf(0x03FF1602, 1, 8, data1);
  
  // Message 2
  unsigned char data2[] = {0xDD, 0xB8, 0xFB, 0x43, 0x3D, 0x70, 0x49, 0xDB};
  // Send message with ID 0x02FF2602 (extended ID)
  CAN.sendMsgBuf(0x02FF2602, 1, 8, data2);
  
  Serial.println("Sent both CAN messages");
  
  delay(2000); // Wait 2 seconds before sending again
}
```

4. Using the table below, wire up your arduino to the MCP2515 module

| **Arduino Pin** | **MCP2515 Pin** | **Purpose**     |
|-----------------|-----------------|-----------------|
|        5V       |       VCC       |      Power      |
|       GND       |       GND       |      Ground     |
|       D10       |        CS       | SPI Chip Select |
|       D12       |        SO       |     SPI MISO    |
|       D11       |        SI       |     SPI MOSI    |
|       D13       |       SCK       |    SPI Clock    |
|        D2       |       INT       |  Interrupt pin  |

5. On the battery base connector cable, connect the small green wire to CAN “H” and connect the blue wire to CAN “L” on the MCP2515
6. Connect the base to the battery and plug in the base cable to the base.
7. Finally plug in your arduino and if everything has gone correctly, you should now be seeing about 48V being outputted on the large red and black wire on the battery base connector :partying_face:
<img src="https://raw.githubusercontent.com/harveyg234/RPH0002-Battery-Arduino-Fix/refs/heads/main/images/Screenshot_180.png" width="50%">

**Note: Once the battery recieves this message once, it will allow a small amount of current through it forever. But once something which draws a high amount of current is connected, the battery will lock again meaning you will be required to use this to keep sending the message.**

The messages Are: **03FF1602#043B8F00002B0412     02FF2602#DDB8FB433D7049DB**

**The Battery talks at 250000kbps CANbus Baudrate**

# If you have the battery and nothing else:
**Note: The battery's connectors(pins) are reversible. For example, if positive is on pin 2, then it will also be on pin 13 ect.**

Use diagram below as reference.
1. Connect the CAN “H” on the MCP2515 to either pins 6 or 9 on the battery
2. Connect the CAN “L” on the MCP2515 to either pins 2 or 13 on the battery
3. Connect your arduino to power and your battery should be outputting 48V on pins + 1, 14 ,   - 4, 11 :partying_face:

![Screenshot_181.png](https://github.com/harveyg234/RPH0002-Battery-Arduino-Fix/blob/main/images/Screenshot_181.png)
