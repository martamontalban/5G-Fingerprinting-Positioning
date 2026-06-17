# Evaluation of 5G Fingerprinting Methods for Robust Position Estimation

Master's Thesis in Telecommunications Engineering – Universidad Carlos III de Madrid (2024–2026).

## Overview

This project investigates fingerprinting-based localization using 5G signal simulations. Two fingerprint methods are compared across regular (LOS-dominated) and irregular (NLOS-rich) urban scenarios using ray tracing with NVIDIA's Sionna library.

### What is Fingerprinting?

Fingerprinting is a localization technique where each physical location is characterized by a unique radio "fingerprint" — a set of signal features measured from nearby base stations. During positioning, a device's measured fingerprint is compared against a pre-built database of known fingerprints, and the closest match reveals the estimated position. Unlike GPS, this approach works in environments where satellite signals fail (tunnels, indoors, urban canyons).

### Methods

- **Method 1:** Channel gain only (one power value per transmitter, in dB).
- **Method 2:** Channel gain + mean delay + RMS delay spread + strongest path delay + strongest path gain (5 features per transmitter).

Position estimation is performed using k-Nearest Neighbors (kNN). Method 1 uses direct Euclidean distance; Method 2 uses z-score normalized Euclidean distance to handle features with different units and scales.

## Scenarios

- **Regular (Plaza Mayor, Madrid):** An open, symmetric square with 4 transmitters at the corners (z=20m). Line-of-Sight dominates. Grid of 234 training receivers, 100 random test receivers.
- **Irregular (Calle Esparteros, Madrid):** A narrow asymmetric street with 2 transmitters (z=5m). Strong NLOS and multipath. Grid of ~198 training receivers, 100 random test receivers.

## Key Results

| Scenario | Condition | Method 1 (best k) | Method 2 (best k) | Improvement |
|----------|-----------|--------------------|--------------------|-------------|
| Regular | LOS | 1.14 m (k=5) | 1.28 m (k=5) | ~ equal |
| Regular | NLOS6 | 3.53 m (k=3) | 3.23 m (k=5) | ~ equal |
| Irregular | LOS | 19.32 m (k=13) | 19.34 m (k=13) | ~ equal |
| Irregular | NLOS1 | 14.92 m (k=13) | 15.79 m (k=13) | ~ equal |
| Irregular | NLOS2 | 6.76 m (k=5) | 5.62 m (k=3) | **17%** |
| Irregular | **NLOS6** | **9.46 m (k=11)** | **5.72 m (k=1)** | **40%** |

### Interpretation

- **Regular scenario (LOS):** Both methods perform equally well (~1–2m error). Channel gain alone captures all spatial information when LOS dominates. Delay features add nothing because multipath is minimal.

- **Irregular scenario (low multipath, LOS/NLOS1):** Both methods struggle equally (~15–20m). With limited multipath, neither channel gain nor delays provide enough spatial discrimination in a complex geometry.

- **Irregular scenario (rich multipath, NLOS2/NLOS6):** Method 2 significantly outperforms Method 1. At max_depth=6, the error drops from 9.46m to 5.72m (40% improvement). Importantly, Method 1 *degrades* as multipath increases (6.76m → 9.46m), while Method 2 *remains stable* (~5.7m). This confirms that delay features effectively capture the spatial structure of multipath propagation.

## Conclusions

1. In symmetric, LOS-dominated environments, **channel gain alone is sufficient** for accurate fingerprinting (~1m accuracy).
2. In complex NLOS environments, **multipath-enhanced fingerprints reduce estimation error by up to 40%**, especially when multipath is rich and spatially diverse.
3. The benefit of Method 2 grows with multipath complexity — the richer the environment, the more spatial information delay features provide.
4. Fingerprint design should **adapt to the scenario**: simple power-based fingerprints for open areas, multipath-feature fingerprints for indoor/urban canyon/tunnel environments.

## Repository Structure

```
├── notebooks/
│   ├── Regular_Scenario_Method1.ipynb
│   ├── Regular_Scenario_Method2.ipynb
│   ├── Irregular_Scenario_Method1.ipynb
│   └── Irregular_Scenario_Method2.ipynb
├── scenes/
│   ├── PlazaMayor2/                  (regular scenario – Mitsuba XML + meshes)
│   └── Escena_Irregular_plazaMayor/  (irregular scenario – Mitsuba XML + meshes)
├── results/
│   └── estimation_errors.xlsx
├── requirements.txt
└── README.md
```

## How to Run

### Requirements

- Docker with NVIDIA GPU support
- NVIDIA drivers ≥ 525

### Setup

```bash
docker run --gpus all -it -p 8888:8888 \
  -v $(pwd):/workspace \
  tensorflow/tensorflow:2.15.0-gpu-jupyter bash

pip install -r requirements.txt
jupyter lab --ip=0.0.0.0 --allow-root --no-browser
```

Open the notebooks and execute cells sequentially. Change `max_depth` in the PathSolver configuration to test different propagation conditions:
- `max_depth=0` → LOS only
- `max_depth=1` → NLOS with 1st-order reflections
- `max_depth=2` → NLOS with 2nd-order reflections
- `max_depth=6` → NLOS with deep multipath

## Future Work

- **Indoor/tunnel scenarios:** Evaluate performance in fully enclosed environments where GPS is unavailable.
- **Machine learning methods:** Replace kNN with neural networks or random forests for potentially better generalization.
- **Adaptive feature selection:** Automatically weight or discard features based on their spatial informativeness per scenario.
- **Dynamic environments:** Extend to mobile users with time-varying channels.
- **Hybrid positioning:** Combine fingerprinting with angle-of-arrival or time-of-arrival measurements.

## Tools Used

- [Sionna RT](https://nvlabs.github.io/sionna/) – Differentiable ray tracing for radio propagation
- [Blender](https://www.blender.org/) + OpenStreetMap – 3D scene generation
- TensorFlow 2.15 – Computational backend
- Python 3.12, NumPy, Matplotlib, scikit-learn

## Author

Marta María Montalbán Luquero

Supervised by Ana García Armada – Universidad Carlos III de Madrid

## License

This work is licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/).
