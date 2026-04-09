# Bearing Fault Diagnosis — Signal Processing based Analysis

**XJTU Gearbox Dataset · 1st Stage Bearing (61800-type DGBB) · 1800 RPM**

A rigorous, fully reproducible pipeline that takes raw two-channel vibration signals from a gearbox test rig and delivers interpretable fault diagnosis — no black box required. The project implements classical signal processing methods that underpin modern condition monitoring systems: filter design, FFT spectral analysis, STFT time-frequency representation and spectral feature engineering.

---

## Results at a Glance

### Signal Preprocessing

The preprocessing pipeline applies a 4th-order Butterworth high-pass filter (20 Hz cutoff) for DC and low-frequency structural noise removal, followed by crest-factor-based outlier rejection. The figure below shows the raw vs. processed time-domain signal alongside the FFT spectrum with class-selective fault-frequency markers, and the crest-factor distribution used for segment quality control.

![Phase 1 Preprocessing — Outer Race](results/phase1_Outer_Race.png)

> *Raw (grey) vs. processed (blue) vibration signal, FFT spectrum with BPFO harmonic markers, and crest-factor distribution after outlier rejection.*

---

### Filter Bank Frequency Responses

Two Butterworth filters are designed and validated before any data is processed. The magnitude and phase responses confirm correct −3 dB cutoffs and the expected attenuation characteristics.

![Filter Bank Responses](results/filter_responses.png)

> *Top row: magnitude response [dB]. Bottom row: phase response. Left: 8th-order anti-alias LP (9216 Hz). Right: 4th-order DC-removal HP (20 Hz).*

---

### Dataset Statistics

After preprocessing and outlier rejection the five fault classes contain a balanced set of 8192-sample segments. The box plots confirm z-score normalisation, and the violin plots show the crest-factor spread per class — elevated crest factor in the Ball and Compound classes is physically consistent with impulsive rolling-element damage.

![Dataset Statistics](results/phase1_dataset_stats.png)

> *Left: segment count per class. Centre: per-segment RMS distribution (Ch1). Right: crest-factor violin plots after outlier rejection.*

---

### FFT Spectral Analysis

Mean amplitude spectra (± 1σ shaded band) are computed from all segments via windowed FFT with Hann taper and amplitude correction. Each subplot overlays only the fault frequencies relevant to that class, preventing visual clutter.

![FFT Per-Class Spectra — Ch1](results/fft_per_class_ch1.png)

> *Per-class mean FFT spectrum (Ch1, 0–1500 Hz). Coloured vertical markers indicate class-selective characteristic frequencies: BPFO/harmonics (amber), BPFI/harmonics (violet), BSF (green), FTF (pink), shaft (grey).*

The overlay plot places all five classes on a single axis to reveal inter-class spectral differences directly.

![FFT Overlay](results/fft_overlay_ch1.png)

> *All five fault classes overlaid. The Normal class shows a smooth broadband floor; fault classes exhibit elevated peaks at their characteristic frequencies.*

---

### STFT Spectrogram Analysis

Short-Time Fourier Transform spectrograms (Hann window, N=512, 75% overlap, Δf ≈ 40 Hz, Δt ≈ 6.25 ms) show how fault energy distributes over time and frequency within a single 400 ms segment. Horizontal dashed lines mark the relevant characteristic frequencies for each class.

![STFT Spectrograms — Ch1](results/stft_per_class_ch1.png)

> *Top row: STFT power spectrogram (inferno colourmap, dB scale). Bottom row: time-averaged spectrum. Horizontal markers show class-selective fault frequencies.*

The resolution trade-off figure demonstrates how window length affects time vs. frequency resolution.

![STFT Resolution Comparison](results/stft_resolution_comparison.png)

> *Left: high time resolution (NFFT=256, Δf=80 Hz). Centre: balanced default (NFFT=512). Right: high frequency resolution (NFFT=2048, Δf=10 Hz).*

---

### Feature Extraction

Eighteen scalar features are extracted per segment per channel across three domains:

| Domain | Features |
|---|---|
| Time domain | RMS, Peak, Crest Factor, Kurtosis, Skewness |
| Spectral (FFT) | Centroid, Bandwidth, Spectral Kurtosis, Peak Frequency, Peak Amplitude, Band Energy Ratios |
| STFT-based | Mean Energy, Energy Variance, Temporal Centre of Gravity |

**Violin + strip plots** show per-class feature distributions and confirm strong class separability for kurtosis, spectral centroid, and band energy ratios.

![Feature Distributions](results/feature_distributions.png)

> *Violin plots (shape = distribution) with strip overlay (individual segments). Features with non-overlapping distributions are the most useful for fault classification.*

**Scatter plot** of kurtosis vs. spectral centroid shows clear cluster separation between Normal, Inner Race, and Outer Race.

![Feature Scatter — Kurtosis vs Spectral Centroid](results/feature_scatter_kurtosis_spectral_centroid.png)

> *Each point is one segment. Clusters confirm that kurtosis and spectral centroid together separate the five fault classes.*

**Feature correlation heatmap** identifies redundant features. Highly correlated pairs (|r| > 0.85) can be pruned before training without information loss.

![Feature Correlations](results/feature_correlations.png)

**Radar chart** normalises all features to [0, 1] and overlays mean profiles per class, making it easy to see which fault type has the highest energy, most impulsive behaviour, or highest-frequency spectral content.

![Feature Radar](results/feature_radar.png)

**Band energy heatmap** shows the fraction of total spectral power in each frequency octave per class. Outer Race energy concentrates in the 100–300 Hz band (consistent with BPFO and its harmonics); Inner Race energy is distributed higher due to BPFI sidebands.

![Band Energy Heatmap](results/band_energy_heatmap.png)

---


## Technical Summary

| Property | Value |
|---|---|
| Bearing type | 61800-type DGBB · Nb=7, Bd=2.1 mm, Pd=14.5 mm, α=0° |
| Shaft speed | 1800 RPM (fr = 30 Hz) |
| Sampling rate | 20 480 Hz |
| Channels | 2 (horizontal + vertical acceleration) |
| Fault classes | Normal · Ball · Inner Race · Compound · Outer Race |
| Segment length | 8192 samples (400 ms) · 50% overlap |
| Preprocessing | HP filter 20 Hz + crest-factor outlier rejection + z-score |
| FFT | Hann window · amplitude-corrected · dB scale |
| STFT | N=512 · 75% overlap · Δf=40 Hz · Δt=6.25 ms |
| Features | 18 per segment per channel |
| Envelope method | SK kurtogram → BP filter → Hilbert transform → FFT |
| Diagnostic metric | Harmonic Sum Score (HSS) + log-ratio classifier |

---

## Characteristic Frequencies

| Frequency | Symbol | Value |
|---|---|---|
| Shaft | fr | 30.000 Hz |
| Cage | FTF | 12.828 Hz |
| Ball spin | BSF | 101.343 Hz |
| Outer race | BPFO | 89.793 Hz |
| Inner race | BPFI | 120.207 Hz |
| Sanity check | BPFO + BPFI = Nb × fr | 210.0 Hz ✓ |

---

## Pipeline Architecture

```
Raw .txt files  (Chan1.txt · Chan2.txt)
        │
        ▼
Cell 1 ── Config + bearing geometry calculator
              Derives BPFO · BPFI · BSF · FTF from Nb, Bd, Pd, α
        │
        ▼
Cell 2 ── Data loading + Phase 1 preprocessing
              HP filter → segmentation → crest-factor rejection → z-score
        │
        ▼
Cell 3 ── FFT analysis
              Windowed FFT → mean/std spectra → class-selective markers
        │
        ▼
Cell 4 ── STFT analysis
              Hann STFT → spectrograms → time-averaged spectra
        │
        ▼
Cell 5 ── Feature extraction
              18 features/segment → DataFrame → distributions · scatter · radar
    
```

---

## Dependencies

```
numpy · pandas · scipy · matplotlib · seaborn · scikit-learn · torch · tqdm
```

```bash
pip install -r requirements.txt
jupyter notebook FMA_Bearing_Fault_Diagnosis.ipynb
```

Run cells in order: Cell 1 → Cell 6.

---

## References

- Antoni J. (2006). *The spectral kurtosis: a useful tool for characterising non-stationary signals*. Mechanical Systems and Signal Processing, 20(2), 282–307.
- Antoni J. (2007). *Fast computation of the kurtogram for the detection of transient faults*. Mechanical Systems and Signal Processing, 21(1), 108–124.
- Randall R. B. & Antoni J. (2011). *Rolling element bearing diagnostics — a tutorial*. Mechanical Systems and Signal Processing, 25(2), 485–520.

---

### If this repository contributes to your research, please consider citing the associated publications by the author.
## Citation

If you use this repository in your research or work, please cite:

@misc{kumar2026bearing,
  author       = {Rajeev Kumar},
  title        = {Bearing Fault Diagnosis using FFT, STFT, and Envelope Spectrum Analysis},
  year         = {2026},
  howpublished = {\url{https://github.com/rajeevfee/bearing_faults_daignosis}},
  note         = {GitHub repository}
}



## License

MIT — free to use, adapt, and cite.
