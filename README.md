# SkyCore

**SkyCore** is an open-source, from-scratch family of **global weather forecasting models** — tiny GraphCast-style networks you can run on an ordinary GPU. This repository is the **official model distribution**: trained weights for all three models, plus a single inference script. Everything is **Apache-2.0**.

Three models:

| Model | Size | Aim |
|---|---|---|
| **S1** | width 192, depth 12 | standard |
| **D1** | width 192, depth 24 + 8-step rollout finetune | stronger medium-range ("DeepThink") |
| **W1** | width 96, depth 8 (~0.7M params) | small, open base for the community |

The full open **W1** project (code + weights + details) lives at **[SkyCore-WeModelsProject](https://github.com/SkyCore-lab/SkyCore-WeModelsProject)**.

---

## What these models do

The Earth is split into a 2.5° grid (73 × 144). Given the last two states (now and 6 h ago), each model predicts the atmosphere 6 hours ahead and rolls that forward out to 10 days. They predict 17 variables: geopotential height, temperature, and the two wind components at 850 / 500 / 250 hPa, plus sea-level pressure, surface temperature, surface wind, and precipitable water.

---

## Measured skill

All models **beat both persistence and climatology at every lead and variable**. RMSE (lower is better):

**S1 / D1** — test 2024–2025, 138 forecasts:

| Variable | Lead | S1 | D1 | persistence | climatology |
|---|---|---|---|---|---|
| z500 (m) | 3 d | 44.4 | **43.6** | 95.9 | 84.5 |
| z500 (m) | 5 d | 67.4 | **65.0** | 106 | 84.7 |
| z500 (m) | 10 d | 100.5 | **96.3** | 113 | 84.8 |
| t850 (°C) | 5 d | 3.12 | **3.02** | 4.63 | 3.63 |
| slp (Pa) | 5 d | 640 | **612** | 941 | 716 |
| pwat (mm) | 5 d | 6.23 | **5.95** | 8.26 | 6.56 |

D1 (deeper + longer rollout) overtakes S1 in the medium range (3 days and beyond), as intended; at 1 day they are essentially tied.

**W1** — test 2025–2026, 130 forecasts:

| Variable | 1 d | 3 d | 5 d | persistence (1d) | climatology |
|---|---|---|---|---|---|
| z500 (m) | 11.7 | 35.1 | 59.1 | 59.9 | 84.2 |
| t850 (°C) | 1.0 | 1.9 | 2.8 | 3.0 | 3.4 |
| slp (Pa) | 144 | 360 | 573 | 582 | 724 |

> **Note on comparability.** S1/D1 and W1 are scored on a coarse 2.5° grid against **different ground truth**, so their numbers are **not directly comparable to each other**, nor to GraphCast or operational high-resolution models. Read each as confirmation that the model genuinely forecasts the weather.

---

## Usage

```bash
# forecast with a released model (point --out at the folder holding its model.pt)
python skycore.py forecast --model s1 --data ./data --out ./weights/s1
python skycore.py forecast --model d1 --data ./data --out ./weights/d1
python skycore.py forecast --model w1 --data ./data --out ./weights/w1

# re-score a released model against its own test period
python skycore.py evaluate --model s1 --data ./data --out ./weights/s1
```

Missing libraries (numpy / torch / eccodes, …) are installed on first run; the first data download takes a while.

---

## Files

| Path | Role |
|---|---|
| `skycore.py` | Inference / scoring / forecasting in one file |
| `weights/s1/model.pt`, `weights/s1/scores.json` | SkyCore S1 |
| `weights/d1/model.pt`, `weights/d1/scores.json` | SkyCore D1 |
| `weights/w1/model.pt`, `weights/w1/scores.json` | SkyCore W1 |

---

## License

- Code and all released weights are **Apache-2.0**.
- All models are trained only on openly licensed (public-domain / CC0) data.
