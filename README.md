````markdown
<div align="center">

<h1>Awesome SmellAI</h1>
<p><em>Real-time capture → ML prediction → 12-channel wearable release.</em></p>

<!-- Badges (replace placeholders as needed) -->
<a href="https://arxiv.org/abs/XXXXX.YYYYY"><img alt="arXiv" src="https://img.shields.io/badge/arXiv-XXXXX.YYYYY-b31b1b.svg"></a>
<a href="https://Alistair0909.github.io/SmellAI/"><img alt="Project Page" src="https://img.shields.io/badge/Project-Website-1f6feb.svg"></a>
<a href="https://huggingface.co/datasets/DeweiFeng/smell-net"><img alt="Dataset" src="https://img.shields.io/badge/Dataset-SMELLNET-green.svg"></a>
<a href="https://github.com/sindresorhus/awesome"><img alt="Awesome" src="https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.10%2B-blue.svg">
<img alt="License" src="https://img.shields.io/badge/License-MIT-black.svg">

<br/><br/>

<!-- Authors -->
<a href="https://github.com/AlistairPernigo" target="_blank">Alistair Pernigo</a> (MIT) •
<a href="https://github.com/KaichenZhou" target="_blank">Kaichen Zhou</a> (MIT) •
<a href="https://github.com/yunge-wen" target="_blank">Yunge Wen</a> (NYU) •
<a href="https://github.com/jasbrooks" target="_blank">Jas Brooks</a> (MIT CSAIL) •
<a href="https://github.com/pliang279" target="_blank">Paul Pu Liang</a> (MIT) •
<a href="https://github.com/DeweiFeng" target="_blank">Dewei Feng</a> (MIT) •
<a href="https://github.com/ddvd233" target="_blank">David Dai</a> (MIT)

</div>

---

**Awesome SmellAI** is an end-to-end pipeline that **captures** odors with a 4-channel e-nose, **predicts** their composition with **ScentRatioNet**, and **releases** reconstructed scents via a **12-channel wearable** (Scentrealm Neckwear) over BLE — all **in real time**.

<p align="center">
  <img src="__assets__/videos/realtime_plot.gif" width="31%" alt="Realtime Capture">
  <img src="__assets__/videos/prediction_bars.gif" width="31%" alt="12-Class Prediction">
  <img src="__assets__/videos/ble_release.gif" width="31%" alt="BLE Release">
</p>

---

## Table of Contents
- [Highlights](#highlights)
- [Gallery](#gallery)
- [Quickstart](#quickstart)
- [Inference (Capture → Predict)](#inference-capture--predict)
- [BLE Release](#ble-release)
- [Training](#training)
- [System Overview](#system-overview)
- [Olfactory Stimulation Approaches in HCI](#olfactory-stimulation-approaches-in-hci)
- [Awesome Olfactory HCI (Curated)](#awesome-olfactory-hci-curated)
- [References](#references)
- [Cite](#cite)
- [Contact](#contact)
- [Contributing & License](#contributing--license)
- [Publish Checklist](#publish-checklist)

---

## Highlights
- **Single flow:** Wio + Grove GM-102/302/502/702 → **ScentRatioNet** (4-ch → 12-D mixture ratios) → **12-ch wearable** via BLE.
- **Human-aligned output:** Normalized weights map **1-to-1** to dispenser channels (with `top-k` filtering & temperature).
- **Portable & fast:** Low-latency live plots for interactive demos.
- **Palette (1–12):** banana, orange, pear, apple, mango, peach, strawberry, clove, coriander, garlic, almond, cumin.

---

## Gallery
Below we illustrate (i) real-time 4-channel capture, (ii) 12-class mixture prediction, and (iii) BLE release. More results and videos are on the **Project Page**.

<table>
  <tr>
    <td><img src="__assets__/videos/realtime_plot.gif" alt="Realtime Capture"></td>
    <td><img src="__assets__/videos/prediction_bars.gif" alt="12-Class Prediction"></td>
    <td><img src="__assets__/videos/ble_release.gif" alt="BLE Release"></td>
    <td><img src="__assets__/videos/mix_demo.gif" alt="Mixture Demo"></td>
  </tr>
  <tr>
    <td colspan="2"><center>“60 s capture @ 10 Hz from Wio + GMXXX (NO₂ / C₂H₅OH / VOC / CO)”</center></td>
    <td colspan="2"><center>“Release via Scentrealm Neckwear — channels mapped from predicted weights”</center></td>
  </tr>
</table>

> Model: **ScentRatioNet** (4-channel time-series → 12-class mixing ratios)

---

## Quickstart
```bash
git clone https://github.com/<YOUR-ORG>/Awesome-SmellAI.git
cd Awesome-SmellAI

# Environment
conda create -n smellai python=3.10 -y
conda activate smellai
pip install -r requirements.txt

# Pretrained models & scalers
mkdir -p models
# Place:
# models/mixture_kl_model.pth
# models/channel_scalers_4.joblib
````

---

## Inference (Capture → Predict)

<details>
<summary><strong>Windows (PowerShell / CMD)</strong></summary>

```bash
python scripts/live_capture_to_weights.py ^
  --port COM7 --baud 115200 --duration 60 ^
  --outdir "data\output" ^
  --label "Orange" ^
  --model-path "models\mixture_kl_model.pth" ^
  --scalers-path "models\channel_scalers_4.joblib" ^
  --win-len 600 --hop-len 300 ^
  --topk 6 --temperature 1.0 ^
  --plot-every 2 --allow-pad
```

</details>

<details>
<summary><strong>macOS / Linux</strong></summary>

```bash
python scripts/live_capture_to_weights.py \
  --port /dev/tty.usbmodem* --baud 115200 --duration 60 \
  --outdir data/output \
  --label Orange \
  --model-path models/mixture_kl_model.pth \
  --scalers-path models/channel_scalers_4.joblib \
  --win-len 600 --hop-len 300 \
  --topk 6 --temperature 1.0 \
  --plot-every 2 --allow-pad
```

</details>

**Artifacts written to**:

* `data/output/predictions.csv` — 12-class probabilities
* `data/output/device_weights_full.json` and `_topk.json` (+ CSV)
* `data/output/plots/` — saved figures

> **Channel map (1–12):** banana, orange, pear, apple, mango, peach, strawberry, clove, coriander, garlic, almond, cumin.

---

## BLE Release

```bash
# 1) List BLE services (verify Nordic UART Service)
python scripts/ble_list_services.py --address 08:F9:E0:E4:57:FE

# 2) Smoke test (single channel + handset gain)
python scripts/ble_smoke_test.py --address 08:F9:E0:E4:57:FE --channel 8 --seconds 8 --handset 230

# 3) Send predicted weights (top-k JSON → timed pulses)
python scripts/ble_send_weights.py \
  --address 08:F9:E0:E4:57:FE \
  --json data/output/device_weights_topk.json \
  --total-seconds 20 --min-per-channel 1 --handset 230
```

> **Tips**
>
> * If nothing releases, try a different `--channel` and increase `--handset` (e.g., 240–255).
> * Verify cartridges aren’t depleted; device is charged and awake.

---

## Training

**Dataset format (10 Hz; clean `baseline → exposure → recovery`):**

```
timestamp_ms,NO2,C2H5OH,VOC,CO
```

**Example config (`configs/smellai.yaml`):**

```yaml
train_data:
  csv_root:  data/train_csv/
  win_len:   600
  hop_len:   300
  normalize: per_channel
model:
  arch:       ScentRatioNet
  n_channels: 4
  n_classes:  12
opt:
  lr: 3e-4
  batch_size: 32
  epochs: 100
```

**Run training:**

```bash
python train.py --config configs/smellai.yaml
```

---

## System Overview

```
[Headspace] → [4-ch MOX Array] → [Standardize + Window] → [ScentRatioNet (12-D ratios)]
                                                    ↓
                                [BLE Scheduler: top-k, τi = pi · T, τi ≥ τmin]
                                                    ↓
                                   [12-ch Wearable: sequential micro-puffs]
```

* **Sensing.** Wio Terminal + Grove Multichannel Gas Sensor v2 (GM-102B/302B/502B/702B).
* **Model.** Transformer-style encoder → normalized mixture ratios (12-D).
* **Actuation.** Non-overlapping pulses proportional to predicted weights; per-channel minimums and global gain.

---

## Olfactory Stimulation Approaches in HCI

| Modality              | Delivery method               | Typical setup/examples                              | Strengths for HCI                    | Main limitations                    | Representative references                  |
| --------------------- | ----------------------------- | --------------------------------------------------- | ------------------------------------ | ----------------------------------- | ------------------------------------------ |
| Chemical              | Pressurized micro-jets        | High-velocity aerosol streams; multi-channel mixing | Good spatial control, short response | Solenoid count; plumbing complexity | Covington 2018; Dobbelstein 2017           |
| Chemical              | Fan-guided airflow            | Fast fan steering; low cost                         | Fast, simple                         | Lower precision, plume spread       | Hartmann 1902; Iwamoto 2009; Matosich 2021 |
| Chemical              | Liquid atomizer               | Aerosolized perfume via ToF chamber                 | Precise channel/plume control        | Needs chamber, heating & cartridges | Kakehi 2007; Lee 2023; Nakamoto 2016; 2012 |
| Chemical              | Ultrasound-driven atomization | Ultrasonic vibration atomization                    | Excellent temporal control           | Complex; limited spatial resolution | Hasegawa 2018                              |
| Chemical              | Electronic micro-vaporization | Micro-vaporizer cartridges                          | Fine-grained dosing                  | Requires custom micro-vaporizer     | Bult 2007; Gerkin 2021                     |
| Chemical/Liquid       | e-nose (sensing)              | MOX, QCM, SAW arrays                                | Rich odor features                   | Sensor drift; humidity              | Conesa 2022; Kratz 2022; Lu 2020; Mu 2020  |
| Chemical              | Gas-phase matrix              | MS-based olfactometer                               | High fidelity                        | Large, expensive                    | Nakamoto 2016; 2012                        |
| Chemical              | Pressure pulses               | Pulsed valves; high-speed injection                 | Sharp onsets                         | Valve wear                          | Tachimoto 2016                             |
| Electrical/Trigeminal | Thermal/chemical nasal        | Localized trigeminal stimuli                        | Millisecond timing                   | Not olfaction per se; safety gating | Brooks 2020; 2021; Heilig 1962             |

---

## Awesome Olfactory HCI (Curated)

> **Preview | Title | Publication | Links**
> Replace images in `assets/img/*.jpg` with your own thumbnails.

### Chemical — Wearables & Ambient Delivery

|                            Preview                           | Title                                                 |          Publication         |                        Links                       |
| :----------------------------------------------------------: | :---------------------------------------------------- | :--------------------------: | :------------------------------------------------: |
|        <img src="assets/img/ezzence.jpg" width="260">        | Ezzence: Modular Scent Wearable for Sleep             | Frontiers in Psychology 2022 | [Paper](https://doi.org/10.3389/fpsyg.2022.791768) |
|        <img src="assets/img/essence.jpg" width="260">        | Essence: Olfactory Interfaces for Mood & Performance  |           CHI 2017           |  [Paper](https://doi.org/10.1145/3025453.3026004)  |
|        <img src="assets/img/inscent.jpg" width="260">        | inScent: Wearable Olfactory Display for Notifications |           ISWC 2017          |  [Paper](https://doi.org/10.1145/3123021.3123035)  |
|     <img src="assets/img/smellomessage.jpg" width="260">     | Smell-O-Message: Olfactory Notifications in Messaging |           ICMI 2018          |  [Paper](https://doi.org/10.1145/3242969.3242975)  |
| <img src="assets/img/portable_multichannel.jpg" width="260"> | Portable, Multichannel Olfactory Transducer           |     IEEE Sensors J. 2018     | [Paper](https://doi.org/10.1109/JSEN.2018.2832284) |

### Chemical — Airstreams, Fans, Ultrasound & Atomizers

|                           Preview                           | Title                                 |     Publication    |                           Links                           |
| :---------------------------------------------------------: | :------------------------------------ | :----------------: | :-------------------------------------------------------: |
| <img src="assets/img/ultrasound_fragrance.jpg" width="260"> | Midair Ultrasound Fragrance Rendering |   IEEE TVCG 2018   |     [Paper](https://doi.org/10.1109/TVCG.2018.2794118)    |
|     <img src="assets/img/back_to_mouth.jpg" width="260">    | Back to the Mouth (gustatory/airflow) | SIGGRAPH 2009 (ET) |      [Paper](https://doi.org/10.1145/1597956.1597960)     |
|  <img src="assets/img/hartmann_odor_show.jpg" width="260">  | “A Trip to Japan in Sixteen Minutes”  |        1902        | [Entry](https://en.wikipedia.org/wiki/Sadakichi_Hartmann) |

### Electrical / Chemosthetic — Trigeminal Stimulation

|                         Preview                        | Title                                              | Publication |                       Links                      |
| :----------------------------------------------------: | :------------------------------------------------- | :---------: | :----------------------------------------------: |
| <img src="assets/img/trigeminal_temp.jpg" width="260"> | Trigeminal-Based Temperature Illusions             |   CHI 2020  | [Paper](https://doi.org/10.1145/3313831.3376806) |
|   <img src="assets/img/stereo_smell.jpg" width="260">  | Stereo-Smell via Electrical Trigeminal Stimulation |   CHI 2021  | [Paper](https://doi.org/10.1145/3411764.3445300) |
|    <img src="assets/img/sensorama.jpg" width="260">    | Sensorama (multisensory cinema incl. scent)        |     1962    | [Entry](https://en.wikipedia.org/wiki/Sensorama) |

### Sensing & Modeling — E-Nose, Sensors, Mass-Spec, Cheminformatics

|                             Preview                             | Title                                       |         Publication        |                         Links                        |
| :-------------------------------------------------------------: | :------------------------------------------ | :------------------------: | :--------------------------------------------------: |
|    <img src="assets/img/low_cost_enose_wine.jpg" width="260">   | Low-Cost E-Nose for Wine Varieties          |        Agronomy 2022       |   [Paper](https://doi.org/10.3390/agronomy12112627)  |
|       <img src="assets/img/whats_cooking.jpg" width="260">      | What’s Cooking? Off-the-Shelf Sensing       |      UIST Adjunct 2022     |   [Paper](https://doi.org/10.1145/3526114.3558687)   |
|        <img src="assets/img/milk_enose.jpg" width="260">        | Milk Source ID & Quality (e-nose + ML)      |        Sensors 2020        |      [Paper](https://doi.org/10.3390/s20154238)      |
|         <img src="assets/img/wine_saw.jpg" width="260">         | Wine Classification with ZnO SAW Array      | Sensors & Actuators B 2006 |  [Paper](https://doi.org/10.1016/j.snb.2006.02.014)  |
|         <img src="assets/img/mems_mox.jpg" width="260">         | Low-Power MOX Sensors with MEMS Heater      |       ACS Omega 2021       |   [Paper](https://doi.org/10.1021/acsomega.0c05532)  |
|         <img src="assets/img/qcm_fruit.jpg" width="260">        | Odor Approximation with QCM                 | Sensors & Actuators B 2007 |  [Paper](https://doi.org/10.1016/j.snb.2006.11.025)  |
|      <img src="assets/img/mass_spec_odors.jpg" width="260">     | Odor Approximation Using Mass Spectrometry  |    IEEE Sensors J. 2012    |  [Paper](https://doi.org/10.1109/JSEN.2012.2190506)  |
|  <img src="assets/img/olfactory_display_book.jpg" width="260">  | Olfactory Display & Odor Recorder (chapter) |         Wiley 2016         | [Chapter](https://doi.org/10.1002/9781118768495.ch7) |
| <img src="assets/img/cheminformatics_creation.jpg" width="260"> | Cheminformatics for Scent Creation          |      Sci. Reports 2024     |  [Paper](https://doi.org/10.1038/s41598-024-82654-7) |
|    <img src="assets/img/principal_odor_map.jpg" width="260">    | Principal Odor Map (POM)                    |        Science 2023        |   [Paper](https://doi.org/10.1126/science.ade4401)   |
|         <img src="assets/img/smellnet.jpg" width="260">         | SMELLNET Dataset                            |         arXiv 2025         |  [Paper](https://doi.org/10.48550/arXiv.2506.00239)  |

### Human Studies, XR & Society

|                            Preview                           | Title                                        |         Publication        |                        Links                       |
| :----------------------------------------------------------: | :------------------------------------------- | :------------------------: | :------------------------------------------------: |
|      <img src="assets/img/quintessence.jpg" width="260">     | QuintEssence: Emotions, Memories, Body Image |         TOCHI 2022         |      [Paper](https://doi.org/10.1145/3526950)      |
| <img src="assets/img/vr_gestures_olfactory.jpg" width="260"> | Mid-Air Gestures for Olfactory VR            |          CHI 2025          |  [Paper](https://doi.org/10.1145/3706598.3713964)  |
|    <img src="assets/img/smell_pittsburgh.jpg" width="260">   | Smell Pittsburgh (citizen science)           |          IUI 2019          |  [Paper](https://doi.org/10.1145/3301275.3302293)  |
|   <img src="assets/img/odor_memory_review.jpg" width="260">  | Odor Memory: Review & Analysis               | Psychon. Bull. & Rev. 1996 |     [Paper](https://doi.org/10.3758/BF03210754)    |
|   <img src="assets/img/thematic_analysis.jpg" width="260">   | Using Thematic Analysis in Psychology        |          QRP 2006          | [Paper](https://doi.org/10.1191/1478088706qp063oa) |

---

## References

A longer reference list is embedded via inline links above. You can also keep a full bibliography under `docs/awesome.md` if you split the curated list in the future.

---

## Cite

If you use this repository, please cite:

```bibtex
@article{Pernigo2025AwesomeSmellAI,
  title   = {Awesome SmellAI: Cracking the Code of Everyday Odors—From Capture to Release},
  author  = {Pernigo, Alistair and Zhou, Kaichen and Wen, Yunge and Brooks, Jas and Liang, Paul Pu and Feng, Dewei and Dai, David},
  year    = {2025},
  journal = {arXiv preprint arXiv:XXXXX.YYYYY}
}
```

---

## Contact

**Alistair Pernigo** — MIT Media Lab — <a href="mailto:jasb@mit.edu">[apernigo@mit.edu](mailto:apernigo@mit.edu)</a>

---

## Contributing & License

* PRs welcome: scripts, datasets, BLE bindings, UX demos. See `CONTRIBUTING.md`.
* MIT License. See `LICENSE`.

```
```


