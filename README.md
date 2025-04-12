# IoT Bit‑plane Compression Demo

This project demonstrates a lightweight, lossless compression framework for continuous IoT sensor data. It leverages FP16 encoding with bit‑plane disaggregation (at the *bit* level), dense bit‑packing, and 4 KB block compression using LZ4 or Zstandard. A Streamlit dashboard provides real‑time visualization of sensor data, detailed compression metrics, network simulation (bandwidth throttle and packet loss), and CSV data export.

## Features

- **Bit‑plane Disaggregation**: Splits each FP16 value into 16 bit‑planes and packs bits with `np.packbits`.
- **4 KB Block Compression**: Each packed plane is split into 4 KB blocks (or a single block if smaller) and compressed.
- **Multi‑Sensor Simulation**: Emulates temperature and humidity data.
- **Real‑Time Dashboard**: Visualizes sensor data, overall and per‑plane compression ratios, latencies, and network conditions.
- **CSV Download**: Export recovered sensor data as CSV.

## Requirements

- Python 3.7+
- Packages: `numpy`, `lz4`, `zstandard`, `streamlit`

## Installation

1. Clone the repository:

   ```bash
   git clone https://github.com/yourusername/iot-bitplane-compression.git
   cd iot-bitplane-compression
   ```

2. Install dependencies:

   ```bash
   pip install numpy lz4 zstandard streamlit
   ```

## Usage

### Running the Sender

Start the sender (which simulates sensor data, converts it to FP16, disaggregates into bit‑planes, packs and compresses the data, then serves it over TCP):

```bash
python pi_offline_sender.py
```

### Running the Dashboard

1. Edit `ui_dashboard.py` and set `PI_HOST` to your sender's IP address.
2. Run the dashboard with Streamlit:

   ```bash
   streamlit run ui_dashboard.py
   ```

Use the sidebar controls to adjust the history window, bit‑planes requested, codec, bandwidth throttle, and packet‑loss simulation. The dashboard displays a live sensor stream, detailed compression statistics, and provides a CSV download option for the recovered data.

A demo video is here https://drive.google.com/file/d/1etdjNjO-VFhYkczx4EUn0cCFB4SHXUjU/view?usp=sharing.

## How It Works

1. **Acquisition & FP16 Conversion**: Sensor data (e.g., temperature and humidity) is sampled and converted to FP16.
2. **Bit‑plane Disaggregation & Packing**: Each FP16 value is split into its 16 bits. Bits from the same position across samples are densely packed.
3. **4 KB Block Compression**: Packed bit‑planes are divided into 4 KB blocks (or one block if smaller) and compressed.
4. **Reconstruction & Visualization**: The dashboard fetches the compressed data, decompresses and unpacks the bits, reconstructs FP16 values, and visualizes the data along with compression metrics.