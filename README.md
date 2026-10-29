<div align="center">

# IronBCI-8

**A wireless brain-computer interface that fits in your hand.**

Low-cost, open-source, wearable 8-channel BCI with Bluetooth LE, a mobile SDK and a Python SDK.

[📖 Documentation](https://docs.pieeg.com/hardware/devices/ironbci/) ·
[🚀 How to Start](https://docs.pieeg.com/hardware/devices/ironbci/setup/) ·
[🌐 Try in the browser](https://cloud.pieeg.com) ·
[💬 Discord](https://discord.gg/5Ps24p47)

<img src="https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_app.jpg" alt="IronBCI-8 with electrodes and the Android app" width="460">

</div>

> [!WARNING]
> IronBCI-8 is **not a medical device** and has not been certified by any regulator for use with the human body. It is a tool for research, education and engineering. You use it at your own risk.

> [!CAUTION]
> **Battery only.** IronBCI-8 runs from its own LiPo battery. **Never wear it while it is charging** over USB. Check the battery **polarity** before connecting: a reversed battery damages the board. Read the [Safety page](https://docs.pieeg.com/hardware/devices/ironbci/safety/) and the [liability notice](https://pieeg.com/liability/).

## At a glance

| | |
|---|---|
| **Channels** | 8 biosignal inputs, plus reference and bias |
| **Converter** | TI ADS1299, 24-bit, simultaneous sampling |
| **Sample rate** | 250 SPS |
| **Signals** | EEG, EMG, ECG, EOG |
| **Microcontroller** | STM32 |
| **Wireless** | Bluetooth Low Energy, appears as `EAREEG` (some units `IronBCI`) |
| **Electrodes** | Dry or wet Ag/AgCl, 2.54 mm header |
| **Battery** | LiPo, 200 mAh recommended, 2.0 mm JST, charged over Micro USB |
| **Software** | [PiEEG Cloud](https://cloud.pieeg.com) (browser), Android app, [PiEEG Server](https://github.com/pieeg-club/PiEEG-server), Python SDK, JavaScript SDK |

## How it works

```
 Electrodes 1–8 ─┐
 Reference (ear) ┼─► ADC board ─► BLE board ─► Bluetooth LE
 Bias (ear) ─────┘   (ADS1299)    (STM32)        │
                         ▲                       ├ cloud.pieeg.com
       LiPo battery ─► Power board               ├ Android app
                                                 ├ PiEEG Server
                                                 └ Python / JS SDK
```

<img src="https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_boards.jpg" alt="The three IronBCI-8 boards: ADC, BLE and power" width="600">

## Quick start

1. **Connect the battery** to the JST connector under the board. **Check the polarity first.**
2. **Charge:** Switch 1 ON, Switch 2 OFF, plug in Micro USB. LED 1 = charging, LED 2 = full. **Unplug the USB** when done.
3. **Connect** 8 electrodes, a reference ear clip and a bias ear clip.
4. **Switch on:** both switches ON. The board starts advertising over Bluetooth.
5. **Connect** with any of these (no pairing in system settings needed):

| Option | How |
|---|---|
| **Browser** (fastest) | Open [cloud.pieeg.com](https://cloud.pieeg.com) in Chrome or Edge → **PiEEG Devices** → **IronBCI** |
| **Android** | Install [`mobile_app/mobile_sdk.apk`](mobile_app), choose **8 ch** in settings |
| **PiEEG Server** | `pip install "pieeg-server[ironbci]"` then `pieeg-server --device ironbci8` |
| **Python** | `pip install bleak matplotlib numpy` then `python SDK/BLE_General/3.Read_data_graph.py` |

When connected, the **BLE LED** turns on and 8 channels stream at 250 samples per second.

Full walkthrough: **[docs.pieeg.com → IronBCI-8 → How to Start](https://docs.pieeg.com/hardware/devices/ironbci/setup/)**

## Battery, switches and LEDs

<img src="https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_polarity.jpg" alt="Battery connector polarity: match the black wire to GND" width="500">

| Mode | Switch 1 (battery) | Switch 2 (data) |
|---|---|---|
| **Recording** | ON | ON |
| **Charging** | ON | OFF |

<img src="https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_switches.jpg" alt="Switch 1 turns on the battery, Switch 2 starts the data flow" width="450">

| LED | Meaning |
|---|---|
| **LED 1** | Charging |
| **LED 2** | Battery fully charged |
| **BLE LED** | Bluetooth connection active |

## Wiring

| Input | Electrode | Purpose |
|---|---|---|
| **Channels 1–8** | EEG electrodes in a cap or headset (10-20 system) | Signal |
| **Reference** | Ear clip on one earlobe | Common 0 µV point for all channels |
| **Bias** | Ear clip on the other earlobe | Cancels common-mode noise (mains hum) |

<img src="https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_connection.png" alt="Electrode, reference and bias connections" width="700">

Suggested positions (from the IronBCI paper): **F7, Fz, F8, C3, C4, T5, Pz, T6**.

## Software

### Python SDK

Scripts built on [`bleak`](https://github.com/hbldh/bleak) in [`SDK/BLE_General`](SDK/BLE_General):

| Script | What it does |
|---|---|
| [`1.Find_Device.py`](SDK/BLE_General/1.Find_Device.py) | Scan and print the device's Bluetooth address |
| [`2.Read_data.py`](SDK/BLE_General/2.Read_data.py) | Connect and print values in µV |
| [`3.Read_data_graph.py`](SDK/BLE_General/3.Read_data_graph.py) | Live graph of all 8 channels |
| [`4.Read_gpaph_Autoscale.py`](SDK/BLE_General/4.Read_gpaph_Autoscale.py) | Live graph with automatic scaling |

```bash
pip install bleak matplotlib numpy scipy
cd SDK/BLE_General
python 1.Find_Device.py
python 3.Read_data_graph.py
```

**BLE protocol:** data characteristic `0000fe42-8e22-4541-9d4c-21edae82ed19` (notify), 8 channels × 3 bytes (24-bit big-endian two's complement) per sample. Details: [Specification](https://docs.pieeg.com/hardware/devices/ironbci/specification/).

### PiEEG Server

```bash
pip install "pieeg-server[ironbci]"
pieeg-server --device ironbci8                                   # scan and connect
pieeg-server --device ironbci8 --ble-address AA:BB:CC:DD:EE:FF   # skip the scan
```

Dashboard at `http://localhost:1617`, plus recording, WebSocket, LSL and VRChat OSC.

<img src="https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_server.png" alt="PiEEG Server dashboard" width="700">

### Android app

Install [`mobile_app/mobile_sdk.apk`](mobile_app) (MIT licence), choose **8 ch** in settings, press **Save** to record and **Stop** to finish, then share the file. The Flutter source is in [`mobile_app/EEG-Flutter-App-master.zip`](mobile_app). Based on [BodyPress](https://play.google.com/store/apps/details?id=com.bodypress.governorhq).

### JavaScript SDK

```js
const eeg = new PiEEG();
await eeg.connectBLE();          // IronBCI-8 is the default; call from a click handler
```

See the [JavaScript SDK docs](https://docs.pieeg.com/software/api/javascript-sdk/).

All options: **[Software docs](https://docs.pieeg.com/hardware/devices/ironbci/software/)**

## Example signals

Recorded with **dry Ag/AgCl electrodes, no gel**, at 250 SPS.

**EEG: alpha rhythm** (eyes closed and open) · [dataset](dataset)

<img src="https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_alpha.png" alt="Alpha rhythm test with wavelet transform" width="700">

**EEG: chewing and blinking artifacts**

<img src="https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_artifacts.png" alt="Chewing and blinking artifacts" width="700">

## Projects

- **Real-time emotion detection with ChatGPT:** [`Chat_GPT/Chat_GPT_Real_time.py`](Chat_GPT) streams EEG, computes alpha power every second and asks an OpenAI model to label it. Guide: [Connect Brain to ChatGPT](https://www.instructables.com/Connect-Brain-to-ChatGPT/).
- **3D-printable headset:** [ironbci_3D_EEG_Printable_Headset](https://github.com/pieeg-club/ironbci_3D_EEG_Printable_Headset).
- **Your own mobile app:** start from the Flutter source in [`mobile_app`](mobile_app).

## Videos

| | | |
|---|---|---|
| [![Device introduction](https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_video_intro.jpg)](https://youtu.be/gWpfsLuq_eE) | [![Mobile SDK app test](https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_video_mobile.jpg)](https://youtu.be/mxDg8fcnsZc) | [![How to program IronBCI](https://raw.githubusercontent.com/pieeg-club/PiEEG-docs/main/public/images/hardware/2026/10/ironbci8_video_sdk.jpg)](https://www.youtube.com/watch?v=iHrR5pZpmRM) |
| Device introduction | Mobile SDK app test | How to program a BCI |

## What's in this repository

| Folder | Content |
|---|---|
| [`SDK/BLE_General`](SDK/BLE_General) | Python BLE SDK: find, read, live graphs |
| [`mobile_app`](mobile_app) | Android app (`.apk`) and Flutter source |
| [`dataset`](dataset) | Alpha test, raw and 8–12 Hz filtered (dry electrodes, 250 SPS) |
| [`Scripts_for_work_with_dataset`](Scripts_for_work_with_dataset) | Band-pass filter and plot recordings |
| [`Chat_GPT`](Chat_GPT) | Real-time emotion detection with an OpenAI model |
| [`3D_headset`](3D_headset) | Link to the 3D-printable headset |
| [`Supplementary files`](Supplementary%20files) | Images used in the original README |

## Documentation

| Page | |
|---|---|
| [IronBCI-8 Intro](https://docs.pieeg.com/hardware/devices/ironbci/) | Overview, comparison with PiEEG-8 |
| [What's in the Box](https://docs.pieeg.com/hardware/devices/ironbci/required-equipment/) | What you need besides the board |
| [How to Start](https://docs.pieeg.com/hardware/devices/ironbci/setup/) | Battery, charging, switches, first connection |
| [Electrodes & Battery](https://docs.pieeg.com/hardware/devices/ironbci/connections/) | Wiring, montage, switches and LEDs |
| [Software](https://docs.pieeg.com/hardware/devices/ironbci/software/) | PiEEG Cloud, Android, PiEEG Server, Python, JavaScript |
| [Signals & Projects](https://docs.pieeg.com/hardware/devices/ironbci/examples/) | Example recordings, dataset, ChatGPT project |
| [Specification](https://docs.pieeg.com/hardware/devices/ironbci/specification/) | Hardware, BLE data format, µV conversion |
| [Troubleshooting](https://docs.pieeg.com/hardware/devices/ironbci/troubleshooting/) | Battery, Bluetooth and signal fixes |
| [Safety](https://docs.pieeg.com/hardware/devices/ironbci/safety/) | Battery-only operation and LiPo care |

## Citation

If you use IronBCI-8 in research, please cite:

```
I. Rakhmatulin. IronBCI: A Low-Cost, Open-Source BCI Platform with Mobile SDK for
Rapid Neurotech Prototyping. Preprints 2025, 2025071198.
DOI: 10.20944/preprints202507.1198.v1
```

## Contact

- 🌐 [pieeg.com](https://pieeg.com/)
- 📖 [docs.pieeg.com](https://docs.pieeg.com/)
- 💬 [Discord](https://discord.gg/5Ps24p47)
- ✉️ [pieeg@pieeg.com](mailto:pieeg@pieeg.com)
