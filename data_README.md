# 📁 Dataset Instructions

## XJTU-SY Gearbox Dataset

This project uses the **XJTU-SY Gearbox Dataset** (Xi'an Jiaotong University).

### Download

1. Visit: https://biaowang.tech/xjtu-sy-gearbox-datasets/
2. Download the bearing vibration data files
3. Place the `.mat` or `.csv` files in this `data/` folder

### Configuration

After downloading, update the `DATA_DIR` path in **Cell 1** of the notebook:

```python
class Config:
    DATA_DIR = "data/"   # ← update this path if needed
```

### Dataset Details

| Property | Value |
|----------|-------|
| Bearing Type | 68100-2RS deep groove ball bearing |
| Shaft Speed | 1800 RPM |
| Sampling Rate | 20480 Hz |
| Fault Classes | Normal, Outer Race, Inner Race, Ball, Cage |
| Channels | 2 (horizontal + vertical acceleration) |

### Citation

If you use this dataset in your research, please cite:
> Wang, B., Lei, Y., Li, N., & Li, N. (2018). A Hybrid Prognostics Approach for Estimating Remaining Useful Life of Rolling Element Bearings. IEEE Transactions on Reliability.
