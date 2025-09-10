<div align="center">
<h3>Awesome SmellAI: Cracking the Code of Everyday Odors – From Capture to Release</h3>

<a href="https://github.com/AlistairPernigo" target="_blank">Alistair Pernigo</a> (MIT) •  <a href="https://github.com/yunge-wen" target="_blank">Yunge Wen</a> (NYU) • <a href="https://github.com/DeweiFeng" target="_blank">Dewei Feng</a> (MIT) • <a href="https://github.com/ddvd233" target="_blank">David Dai</a> (MIT) • <a href="https://github.com/KaichenZhou" target="_blank">Kaichen Zhou</a> (MIT) • <a href="https://github.com/jasbrooks" target="_blank">Jas Brooks</a> (MIT CSAIL) • <a href="https://github.com/pliang279" target="_blank">Paul Pu Liang</a> (MIT)  Dewei Feng David Dai

<br/><br/>

<a href="#"><img alt="arXiv" src="https://img.shields.io/badge/arXiv-XXXXX.YYYYY-b31b1b.svg"></a> <a href="#"><img alt="Dataset" src="https://img.shields.io/badge/Dataset-<SmellAI>-green.svg"></a> <a href="#"><img alt="Project">[![Project Page](https://img.shields.io/badge/Project-Website-1f6feb.svg)](https://Alistair0909.github.io/SmellAI/) </a>

This repository is the official implementation of <b>Awesome SmellAI</b>, an end-to-end pipeline that <b>captures</b> odors with an e-nose, <b>predicts</b> their composition with a ML model (<i>ScentRatioNet</i>), and <b>releases</b> reconstructed scents via a 12-channel wearable (Scentrealm Neckwear) over BLE — all in real time.

</div>

## Gallery

Below we illustrate (i) real-time 4-channel capture, (ii) 12-class mixture prediction, and (iii) BLE release.
More results and videos are on our **Project Page**.

<table>
  <tr>
    <td><img src="__assets__/videos/realtime_plot.gif" alt="Realtime Capture"></td>
    <td><img src="__assets__/videos/prediction_bars.gif" alt="12-Class Prediction"></td>
    <td><img src="__assets__/videos/ble_release.gif" alt="BLE Release"></td>
    <td><img src="__assets__/videos/mix_demo.gif" alt="Mixture Demo"></td>
  </tr>
  <tr>
    <td colspan="2"><center>"60 s capture @ 10 Hz from Wio + GMXXX (NO₂ / C₂H₅OH / VOC / CO)"</center></td>
    <td colspan="2"><center>"Release via Scentrealm Neckwear — channels mapped from predicted weights"</center></td>
  </tr>
</table>

Model: <b>ScentRatioNet</b> (4-channel time-series → 12-class mixing ratios)

## Steps for Inference

### Prepare Environment

```bash
git clone https://github.com/<YOUR-ORG>/Awesome-SmellAI.git
cd Awesome-SmellAI

# Conda (Windows/Linux/macOS)
conda create -n smellai python=3.10 -y
conda activate smellai

# Python deps
pip install -r requirements.txt
# or (minimal)
# pip install numpy pandas matplotlib joblib pyserial torch torchvision torchaudio bleak crcmod
```

### Download / Place Pretrained Files

```bash
# Put your model + scalers here:
# Awesome-SmellAI/
# └─ models/
#    ├─ mixture_kl_model.pth
#    └─ channel_scalers_4.joblib
mkdir -p models
# (Copy your .pth and .joblib into the models/ folder)
```

### Generate Weights (capture → predict)

```bash
# Windows example (COM7; 60 s capture; live plot; end-to-end prediction)
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

This command:

* reads serial CSV (`timestamp_ms,NO2,C2H5OH,VOC,CO`) at 10 Hz,
* shows a **real-time plot**,
* windows and standardizes,
* runs **ScentRatioNet** and saves:

  * `data/output/predictions.csv` (12-class probabilities),
  * `data/output/device_weights_full.json` and `device_weights_topk.json` (+ CSV),
  * plots under `data/output/plots/`.

> The 12 channels are:
> **1 banana, 2 orange, 3 pear, 4 apple, 5 mango, 6 peach, 7 strawberry, 8 clove, 9 coriander, 10 garlic, 11 almond, 12 cumin**.

## Steps for BLE Release (Scentrealm Neckwear)

### Discover & Smoke Test

```bash
# Optional: list services (check Nordic UART Service present)
python scripts/ble_list_services.py --address 08:F9:E0:E4:57:FE

# Quick “does it release?” test (try a few channels & handset gains)
python scripts/ble_smoke_test.py --address 08:F9:E0:E4:57:FE --channel 8 --seconds 8 --handset 230
```

### Send Predicted Weights (top-k → seconds)

```bash
# Use the top-k JSON produced by inference (map weights → seconds; min per channel)
python scripts/ble_send_weights.py ^
  --address 08:F9:E0:E4:57:FE ^
  --json "data\output\device_weights_topk.json" ^
  --total-seconds 20 --min-per-channel 1 --handset 230
```

> Tips:
>
> * If nothing releases, try a different **channel** and higher **--handset** (e.g., 240–255).
> * Ensure cartridges aren’t depleted; device should be charged and awake.

## Steps for Training

### Prepare Dataset

* Collect raw streams @10 Hz for each base (12 classes) and mixtures (e.g., 50/50, 20/80, 33/33/33).
* Export to CSV (the same 4 columns). Organize by label.

Optional auto-captioning for app concepts is unrelated to model training. For odor modeling, ensure clean baseline/exposure/recovery segments per recording.

### Configuration

Edit your training config (example):

```yaml
train_data:
  csv_root:        data/train_csv/
  win_len:         600
  hop_len:         300
  normalize:       per_channel
model:
  arch:            ScentRatioNet
  n_channels:      4
  n_classes:       12
opt:
  lr:              3e-4
  batch_size:      32
  epochs:          100
```

### Training

```bash
python train.py --config configs/smellai.yaml
```

## Contact Us

**Alistair Pernigo** — MIT Media Lab
**Kaichen Zhou** — MIT
**Yunge Wen** — NYU
**Jas Brooks** — MIT CSAIL — <a href="mailto:jasb@mit.edu">[jasb@mit.edu](mailto:jasb@mit.edu)</a>
**Paul Pu Liang** — MIT — <a href="mailto:ppliang@mit.edu">[ppliang@mit.edu](mailto:ppliang@mit.edu)</a>

## Acknowledgements

We build upon open-source communities in time-series learning, embedded sensing, and BLE tooling. Hardware release uses Scentrealm Neckwear’s Nordic UART Service for BLE communication.

## BibTeX

```bibtex
@article{Pernigo2025AwesomeSmellAI,
  title   = {Awesome SmellAI: Cracking the Code of Everyday Odors – From Capture to Release},
  author  = {Pernigo, Alistair and Zhou, Kaichen and Wen, Yunge and Brooks, Jas and Liang, Paul Pu},
  journal = {Proceedings of the ACM on Human-Computer Interaction},
  year    = {2025},
  note    = {arXiv:XXXXX.YYYYY}
}

```

---

> **Placeholders to replace before publishing**
>
> * arXiv ID, dataset URL, project website URL.
> * GitHub profile links for each author (I pre-filled common handles; adjust if needed).
> * Model/scaler filenames if you use different names.
> * Training config path(s) if you publish training code.
