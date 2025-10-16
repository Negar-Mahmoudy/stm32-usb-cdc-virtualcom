# stm32-usb-cdc-virtualcom

This is a project on USB CDC ( Communication Device Class) using the STM32 microcontroller.
Through CDC it is possible to use the STM32 as a virtual COM port, and when connected to a PC/Mac, it appears as another serial interface just like any other UART.

---

## What is USB CDC?
USB CDC (Communication Device Class) is a USB class that is predominantly used for serial bridging over USB.
With this feature:
- Your STM32 board can appear on a PC as a Virtual COM Port (VCP).
- With Terminal applications such as Hercules,you can send and receive data.
- Not external USB to UART converter is required.

---

## main.c 

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

The `while(1)` sends data every 1 second:

```c
while (1)
{
    CDC_Transmit_FS((uint8_t*)data, strlen(data));
    HAL_Delay(1000);
}
```

---

## usbd_cdc_if.c 

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

The incoming USB data is copied into a user-defined buffer (`buffer[32]` in `main.c`). Then the `Buf` is cleared after use.This allows processing data safely.
Note: The **transmit function** works immediately since the application provides the data. But the **receive function** requires user handling because the library cannot assume how to manage data.

---

## Applications

* Debugging without UART hardware.
* Sending sensor data to PC.
* Creating custom PC-to-STM32 protocols over USB.
* Replacing USB-to-UART converters with a direct USB CDC connection.

---
![Device manager](https://github.com/Negar-Mahmoudy/stm32-usb-cdc-virtualcom/blob/main/images/1.png?raw=true)

![Hercules output](https://github.com/Negar-Mahmoudy/stm32-usb-cdc-virtualcom/blob/main/images/2.png?raw=true)

