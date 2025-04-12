# IoT-Bit-plane-Compression-Demo
ECSE 6660: Internetworking of Things

This repository demonstrates a lightweight, lossless compression framework for continuous IoT sensor data. The system leverages FP16 encoding combined with bit‑plane disaggregation and per‑plane dictionary compression. It is designed for resource‑constrained devices (e.g. Raspberry Pi) and includes a real‑time dashboard to monitor performance metrics such as overall and per‑plane compression ratios, transmission latency, and decompression latency.

## Table of Contents

- [IoT-Bit-plane-Compression-Demo](#iot-bit-plane-compression-demo)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Features](#features)
  - [Requirements](#requirements)
  - [Installation](#installation)
  - [Usage](#usage)
    - [Running the Sender](#running-the-sender)
    - [Running the Dashboard](#running-the-dashboard)
  - [Project Structure](#project-structure)
  - [How It Works](#how-it-works)
  - [Future Work](#future-work)

## Overview

In this project, sensor readings are simulated on a Raspberry Pi. The sender script, `pi_offline_sender.py`, converts raw sensor data into 16-bit half-precision floating-point (FP16) numbers, then disaggregates each sample into individual bit‑planes. Each bit‑plane is densely packed (using NumPy’s `packbits` so that eight bits are stored per byte) and segmented into 4 KB blocks. These blocks are then compressed using either LZ4 (default) or Zstandard and served over a TCP connection.

The receiver component, `ui_dashboard.py`, is built with Streamlit. It fetches the compressed data from the Pi, decompresses and reassembles the FP16 values at the bit level, and displays real‑time sensor trends along with detailed compression metrics. The dashboard also simulates network conditions such as packet loss and bandwidth throttling, and offers CSV download capability.

## Features

- **Bit‑plane Disaggregation**: Splits each FP16 sensor reading into 16 separate bit‑planes at the bit level.
- **Dense Bit Packing**: Uses `np.packbits` to store 8 bits per byte, avoiding unnecessary padding.
- **4 KB Block Compression**: Compresses each bit‑plane in 4 KB blocks. If a plane’s packed size is less than 4 KB, a single block is used.
- **Multiple Compression Algorithms**: Supports both LZ4 and Zstandard for fast and efficient compression.
- **Multi‑Sensor Simulation**: Emulates multiple sensors (temperature and humidity by default).
- **Real‑time Dashboard**: A Streamlit dashboard displays live sensor data, overall and per‑plane compression ratios, latencies, and other network statistics.
- **CSV Download**: Users can download the reconstructed sensor data as a CSV file.
- **Simulated Network Conditions**: Includes parameters to simulate bandwidth limits and packet loss.

## Requirements

- **Hardware**: A Raspberry Pi (or any other compatible IoT device for the sender) with network connectivity.
- **Python**: Version 3.7 or later.
- **Dependencies**: Listed in [requirements.txt](#requirements-file) or install manually.

## Installation

1. **Clone the repository:**

   ```bash
   git clone https://github.com/yourusername/iot-bitplane-compression.git
   cd iot-bitplane-compression
   ```

2. **Install dependencies:**

   You can install the required packages using pip:

   ```bash
   pip install numpy lz4 zstandard streamlit
   ```

   Alternatively, use a [requirements.txt](#requirements-file) file if provided:

   ```bash
   pip install -r requirements.txt
   ```

## Usage

### Running the Sender

The sender script simulates sensor data, performs FP16 conversion, bit‑plane disaggregation, dense packing, and compression. It then serves the compressed data over TCP.

To run the sender on your Raspberry Pi (or local machine for testing):

```bash
python pi_offline_sender.py
```

The sender listens on port `50007` (configurable in the script).

### Running the Dashboard

The dashboard script is a Streamlit application that fetches compressed data from the sender, decompresses it, and displays real‑time visualizations and statistics. To run the dashboard:

1. **Update the IP address:**

   Edit `ui_dashboard.py` and set the variable `PI_HOST` to the IP address of your sender device.

2. **Run the Streamlit app:**

   ```bash
   streamlit run ui_dashboard.py
   ```

3. **Interacting with the Dashboard:**

   - Use the sidebar controls to set the history window, requested bit‑planes, codec (LZ4 or Zstandard), simulated bandwidth throttle, and packet loss.
   - The dashboard will show a live chart of sensor values, detailed compression metrics (overall compression ratio, per‑plane ratios, compression latencies, network RTT, and decompression latency), and a CSV download button for the recovered sensor data.

## Project Structure

```
├── pi_offline_sender.py   # Raspberry Pi sender script (data acquisition, FP16 conversion, bit‑plane compression, TCP server)
├── ui_dashboard.py        # Streamlit dashboard for real‑time visualization & statistics, TCP client, decompression
├── README.md              # This file
└── requirements.txt       # (Optional) List of required packages
```

## How It Works

1. **Data Acquisition & FP16 Conversion**:  
   Sensor readings (temperature, humidity, etc.) are simulated and converted to FP16 format using IEEE 754 half‑precision.

2. **Bit‑plane Disaggregation & Dense Packing**:  
   Each FP16 value (16 bits) is split into its constituent bits. All bits in the same bit position across a batch are densely packed using `np.packbits` so that eight bits are stored per byte.

3. **4 KB Block Compression**:  
   The packed bit‑plane is segmented into 4 KB blocks (or a single block if it’s smaller than 4 KB) and compressed using LZ4 or Zstandard. Metadata is recorded in a header that includes the sizes of each block and the exact number of original bits.

4. **Network Transfer**:  
   The sender streams the header and compressed payload over a TCP connection.

5. **Decompression & Reassembly**:  
   The receiver downloads the data, decompresses each block, unpacks the bits (using `np.unpackbits`), trims any padding, and reassembles the FP16 values by bit‑wise OR’ing the bits (shifted to their correct positions).

6. **Visualization**:  
   The Streamlit dashboard visualizes the recovered sensor data in real‑time and displays detailed statistics, including per‑plane and overall compression ratios, transmission and decompression latencies, and other network conditions.

## Future Work

- **Enhanced Sensor Support**: Integrate additional sensors and real‑hardware interfaces.
- **Adaptive Bit‑Plane Selection**: Implement dynamic algorithms to decide which bit‑planes are most critical, based on data variability.
- **Error Correction**: Add support for transmission error correction.
- **Scalability**: Explore scalability with larger batch sizes, multiple senders, or cloud integration.
