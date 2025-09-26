# stm32-usb-cdc-virtualcom

This project demonstrates how to use the **USB CDC (Communication Device Class)** on an STM32 microcontroller.  
By enabling CDC, the STM32 device can act as a **Virtual COM Port** and communicate with a PC over USB just like a regular UART.

---

## What is USB CDC?
USB CDC (Communication Device Class) is a USB class that emulates serial communication over USB.  
With this feature:
- Your STM32 board can appear on a PC as a **Virtual COM Port (VCP)**.
- You can send and receive data using terminal applications such as Hercules.
- It eliminates the need for an external USB-to-UART converter.

---

## Project Structure
The main files involved are:

- **`main.c`**  
  - Initializes the USB device.  
  - Periodically transmits a test string (`"Heloooooooooo\n"`) to the PC.  

- **`usbd_cdc_if.c`**  
  - Provides the interface between USB hardware and user application.  
  - Contains callbacks for **data transmission** and **data reception**.  
  - Handles custom buffer management for received data.

---

## main.c Explained

Key points from `main.c`:

```c
/* Include section */
#include "usb_device.h"
#include "usbd_cdc_if.h"
#include "string.h"

/* Transmit data */
char* data = "Heloooooooooo\n";

/* Buffer for received data */
uint8_t buffer[32];
````

The `while(1)` loop continuously sends data every 1 second:

```c
while (1)
{
    CDC_Transmit_FS((uint8_t*)data, strlen(data));
    HAL_Delay(1000);
}
```

This demonstrates **sending data from STM32 to PC**.

---

## usbd_cdc_if.c Explained

This file contains the implementation of the USB CDC interface.

### 1. Buffers

```c
uint8_t UserRxBufferFS[APP_RX_DATA_SIZE];  // RX buffer
uint8_t UserTxBufferFS[APP_TX_DATA_SIZE];  // TX buffer
```

### 2. Transmit Function

The function `CDC_Transmit_FS()` is used in `main.c` to send data:

```c
uint8_t CDC_Transmit_FS(uint8_t* Buf, uint16_t Len);
```

### 3. Receive Function

The `CDC_Receive_FS()` callback is triggered automatically when data is received from the PC:

```c
static int8_t CDC_Receive_FS(uint8_t* Buf, uint32_t *Len)
{
    USBD_CDC_SetRxBuffer(&hUsbDeviceFS, &Buf[0]);
    USBD_CDC_ReceivePacket(&hUsbDeviceFS);

    memset(buffer, '\0', 32);
    uint8_t len = (uint8_t) *Len;
    memcpy(buffer, Buf, len);
    memset(Buf, '\0', len);

    return (USBD_OK);
}
```

What happens here:

* The incoming USB data is copied into a user-defined buffer (`buffer[32]` in `main.c`).
* The `Buf` is cleared after use.
* This allows you to safely process received data elsewhere in your application.

---

## How Transmission and Reception Work

* **Transmission (TX):**

  * Simple API: `CDC_Transmit_FS(data, length)`
  * You choose the data to send; Cube library handles the transfer.

* **Reception (RX):**

  * Callback-driven: `CDC_Receive_FS()` is called automatically.
  * User must copy/manage the data (because CubeMX cannot know how you want to use it).

---

## How to Test

1. Flash the project onto your STM32 board.
2. Connect the board via USB to your PC.
3. The PC will recognize it as a **Virtual COM Port** (VCP).
4. Open a serial terminal (e.g. Hercules).
5. Set the correct COM port and baud rate (default is irrelevant, since CDC is USB).
6. You should see `"Heloooooooooo"` printed every 1 second.
7. If you send data from the terminal, it will be received in the `buffer[32]` array.

![Device manager](https://github.com/Negar-Mahmoudy/stm32-usb-cdc-virtualcom/blob/main/images/1.png?raw=true)

![Hercules output](https://github.com/Negar-Mahmoudy/stm32-usb-cdc-virtualcom/blob/main/images/2.png?raw=true)

---

## Applications

* Debugging without UART hardware.
* Sending sensor data to PC.
* Creating custom PC-to-STM32 protocols over USB.
* Replacing USB-to-UART converters with a direct USB CDC connection.

---

## Notes

* The **transmit function** works immediately since the application provides the data.
* The **receive function** requires user handling because the library cannot assume how to manage your data.

