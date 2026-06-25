# enose-ghee

Detecting adulterated ghee with a gas sensor array and TinyML.

Motivated by the 2024 Tirupati laddu controversy, where claims of animal fat being used in sacred prasadam sparked a national conversation, this project asks a simple question: can a cheap sensor array and a small neural network do what normally requires a GC-MS lab?

Short answer: well enough to be interesting.

---

## What it does

A 4-channel metal oxide gas sensor (Seeed Grove Multichannel Gas Sensor V2) captures CO, NO2, ethanol, and VOC signatures from heated ghee samples. That data goes to Edge Impulse, where a small dense neural network classifies the sample as pure or adulterated.

86.6% accuracy distinguishing pure ghee from ghee adulterated with coconut oil (10:1 ratio). The quantized int8 model fits in 15.8K flash and 1.4K RAM at 1ms latency on a Cortex-M4F, small enough to run on hardware you can buy for $5.

---

## Hardware

- Arduino Mega 2560
- Seeed Grove Multichannel Gas Sensor V2 (GM102B, GM302B, GM502B, GM702B)
- TFT display (live sensor readout during collection)

---

## How data was collected

`groove.ino` reads all four sensor axes at ~3.3 Hz and streams values over Serial. Samples were collected at 45-65°C for approximately 5 minutes per class. Data forwarded to Edge Impulse via the data forwarder CLI.

Two classes:
- `PureGhee` — home-clarified butter
- `NotSoPureGhee` — pure ghee cut with coconut oil at 10:1

---

## Model

Built and trained entirely in Edge Impulse. Pipeline: raw time-series → Flatten DSP block → dense classifier (64 → 50 → 20 neurons, dropout 0.3) → 2-class output.

| Metric | float32 | int8 |
|---|---|---|
| Accuracy | 86.6% | 83.8% |
| ROC AUC | 0.8733 | 0.8333 |
| Flash (Cortex-M4F) | — | 15.8K |
| RAM | — | 1.4K |
| Inference latency | — | 1ms |

View the public Edge Impulse project: https://studio.edgeimpulse.com/public/581592/latest

---

## What's in this repo

```
groove.ino    ← sensor readout + TFT display (data collection sketch)
```

The trained model and dataset live in Edge Impulse (link above). The paper is a draft and not yet published.

---

## What's next

Inference ran server-side via Edge Impulse during this project. The natural next step is deploying the quantized model directly onto an ESP32 or STM32 and cutting the cloud dependency entirely. The model is small enough.
