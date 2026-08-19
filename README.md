# STM32 3-Channel DIY Oscilloscope

A simple 3-channel digital oscilloscope based on the STM32F207ZG and its three built-in ADCs.
The project uses the STM32F207's ADCs in **Triple Regular Simultaneous Mode** to acquire three analog signals at the same time. The captured data is transferred to a PC over UART and displayed using a Python application.

This project is intended primarily as an educational and experimental oscilloscope rather than a replacement for a commercial oscilloscope.

## Features

- 3 simultaneously sampled analog channels
- STM32F207ZG MCU
- Three 12-bit ADCs
- Triple Regular Simultaneous ADC mode
- Up to 2 MSPS sampling rate per channel
- 1024 samples per channel per acquisition
- Adjustable acquisition time window
- Rising-edge trigger
- Selectable trigger channel
- UART data transfer at 921600 baud
- Python-based PC front end
- Real-time display of all three channels
- Time-window control from the Python GUI


## Repository structure
stm32osc3ch/
├── firmware/
│   ├── Core/
│   ├── Drivers/
│   ├── OSCILLOSCOPE_V2_FINAL.ioc
│   └── ...
├── software/
│   └── oscilloscope3_ch
│   └── requirements.txt
├── README.md
└── LICENSE
License

## Hardware

The firmware is designed for the:

- **NUCLEO-F207ZG**
- MCU: **STM32F207ZGT6**

The current CubeMX configuration uses:

- ADC1: `ADC_CHANNEL_6`
- ADC2: `ADC_CHANNEL_10`
- ADC3: `ADC_CHANNEL_12`
- ADC resolution: 12 bit
- ADC mode: Triple Regular Simultaneous
- ADC sampling time: 3 cycles
- Timer trigger: TIM2
- UART: USART3
- UART baud rate: 921600

The CubeMX project is included in:

`firmware/OSCILLOSCOPE_V2_FINAL.ioc`

## Sampling Modes

The oscilloscope supports five acquisition modes:

| Mode | Sampling rate | Acquisition window |
|------|---------------:|-------------------:|
| 1    | 2 MHz          | ~512 µs            |
| 2    | 200 kHz        | ~5.12 ms           |
| 3    | 20 kHz         | ~51.2 ms           |
| 4    | 2 kHz          | ~512 ms            |
| 5    | 204 Hz         | ~5.02 s            |

The last mode uses 204 Hz rather than exactly 200 Hz, so its acquisition window is approximately 5.02 seconds.

## Data Format

Each packet sent from the STM32 to the PC has the following structure:

+------------+------+----------------+----------------+----------------+
| Header     | Mode | ADC1           | ADC2           | ADC3           |
| 2 bytes    | 1 B  | 1024 samples   | 1024 samples   | 1024 samples   |
+------------+------+----------------+----------------+----------------+

Each ADC sample is transmitted as a 16-bit value. The Python front end splits the packet into three 1024-sample arrays and converts the ADC readings to voltage using a 3.3 V reference and a 12-bit ADC range.

## Trigger

The firmware separates the interleaved ADC data into three individual buffers. The trigger function can then search for a rising edge in one selected channel.
The trigger channel is selected in the firmware using:

#define TRIGGER_CHANNEL

Once a trigger point is found, the samples from all three channels are transmitted relative to that trigger position.

#PC Software

The PC application is written in Python and uses:

PySerial for UART communication
NumPy for data processing
Matplotlib for plotting

The application displays the three ADC channels in real time and automatically updates the horizontal time scale when the sampling mode changes.
Two buttons allow the user to switch between faster and slower acquisition modes.

#Installation
1. Install Python
Python 3.9 or newer is recommended.

2. Install dependencies
From the software/ directory:
pip install -r requirements.txt

3. Configure the serial port

Open:
software/oscilloscope3_ch

and change:
PORT = "/dev/cu.usbmodemXXXXX" for macOS.

For Windows, use: PORT = "COMxx"

For example:

PORT = "COM5"
4. Run the application
python oscilloscope3_ch
Firmware

The firmware project was generated using STM32CubeMX / STM32CubeIDE and uses the STM32F2 HAL.

Open:

firmware/OSCILLOSCOPE_V2_FINAL.ioc

in STM32CubeIDE / STM32CubeMX.

The project targets:

STM32F207ZGTx
and uses the STM32Cube F2 firmware package.

Build the project and flash the resulting firmware to the NUCLEO-F207ZG.

## Limitations

This is an experimental oscilloscope and has several limitations.

-Analog front end

The ADC inputs are directly dependent on the analog front-end circuitry.
Input impedance, input protection, voltage range, bandwidth and signal conditioning are not fixed by the firmware.
A suitable buffer/amplifier stage should be used when driving the STM32 ADC inputs from high-impedance sources.

-UART bandwidth

The captured data is transferred over a 921600-baud UART connection.
A complete three-channel packet contains 6147 bytes, which limits the theoretical update rate to roughly 15 packets per second before accounting for processing and other overhead.

-ADC accuracy

The ADCs have normal device-to-device variations. Small differences between the three channels can therefore occur.
For applications requiring better channel-to-channel accuracy, individual calibration and software compensation may be required.

-No packet checksum

The current protocol does not include a checksum or CRC.
A future version could add one to detect corrupted UART packets.



This project is released under the MIT License.

See LICENSE for details.

## Disclaimer

This project is provided for educational and experimental purposes.
It is not intended to be used as a safety-critical measurement instrument.
