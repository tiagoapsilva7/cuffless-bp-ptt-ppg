# Data

The dataset is **not** redistributed in this repository. Download it from
PhysioNet and place the files here yourself.

## 1. Get the dataset

Pulse Transit Time PPG Dataset, version 1.1.0, open access on PhysioNet:
https://physionet.org/content/pulse-transit-time-ppg/1.1.0/

Either download the ZIP from that page, or use the command line:

```bash
wget -r -N -c -np https://physionet.org/files/pulse-transit-time-ppg/1.1.0/
```

## 2. Place the files

Both notebooks read the CSV distribution that ships inside the dataset, in its
`csv/` folder. Copy that folder here so the tree looks like this:

```
data/
└── csv/
    ├── subjects_info.csv
    ├── s1_sit.csv
    ├── s1_walk.csv
    ├── s1_run.csv
    ├── s2_sit.csv
    └── ...            (22 subjects x 3 activities)
```

Only the `*_sit.csv` recordings and `subjects_info.csv` are used. The `walk` and
`run` recordings can stay in place, they are simply ignored.

## 3. What is read from each file

`subjects_info.csv` provides one row per record with the columns used as labels
and demographics:

| Column | Use |
|:--|:--|
| `record` | matched against `<subject>_sit` |
| `age`, `height`, `weight`, `gender` | demographic features |
| `bp_sys_start`, `bp_dia_start` | cuff reading before the recording |
| `bp_sys_end`, `bp_dia_end` | cuff reading after the recording |

Each `s<N>_sit.csv` provides the raw PPG channels sampled at 500 Hz:

| Column | Site | Wavelength |
|:--|:--|:--|
| `pleth_1` | Distal | Infrared |
| `pleth_2` | Distal | Red |
| `pleth_3` | Distal | Green |
| `pleth_4` | Proximal | Infrared |
| `pleth_5` | Proximal | Red |
| `pleth_6` | Proximal | Green |

## Using the WFDB files instead

If you downloaded the WFDB records (`.dat` / `.hea`) rather than the CSV folder,
convert them once with:

```python
import wfdb, pandas as pd, pathlib

out = pathlib.Path("csv")
out.mkdir(exist_ok=True)
for rec in pathlib.Path("wfdb").glob("*.hea"):
    r = wfdb.rdrecord(str(rec.with_suffix("")))
    pd.DataFrame(r.p_signal, columns=r.sig_name).to_csv(out / f"{rec.stem}.csv", index=False)
```
