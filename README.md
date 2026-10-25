# Khazain Al-Hikma

**A cursive Arabic handwriting corpus from Islamic heritage.**

Khazain Al-Hikma contains 20 historical Arabic manuscripts from the Central Library of the Islamic University of Madinah. The manuscripts date from 979 to 1731 CE and are written in Naskh, Ruq'ah, and Muhaqqaq scripts. Every page has a manual transcription. The dataset also includes automatically segmented line images aligned to those transcriptions, plus a train/val/test split. It is intended for training and benchmarking OCR / HTR models on historical handwritten Arabic.

| | |
|---|---|
| Manuscripts | 20 |
| Scans (two facing pages each) | 1,113 |
| Transcribed pages | 2,216 |
| Transcribed words | ~414,000 |
| Illegible-word markers (`*`) | ~2,630 |
| Line images with transcription | 33,074 |
| Date range | 979 – 1731 CE |

## Repository layout

```
raw/
  MANUSCRIPT_01/
    images/          00000001.tif, 00000002.tif, ...      (one scan = two facing pages)
    annotations/     00000001_part_01.txt, 00000001_part_02.txt, ...
  ...
  MANUSCRIPT_20/
page-level/
  MANUSCRIPT_01/     00000001_right.png, 00000001_left.png, ...
  ...
  page_level_text.csv
  page_level.csv
line-level/
  Manuscript 01/
    segmented_lines/ manuscript_01_00000001_part_01_1.jpg, ...
    manuscript_01.csv
  ...
  dataset.csv
```

Page numbers are eight-digit, zero-padded integers. **In every manuscript they start at `00000001`.**

### `raw/`: scans and ground truth

- **`images/`**: the original high-resolution TIFF scans. Each scan shows two facing pages. (`MANUSCRIPT_16/images/00000086.png` is the only non-TIFF scan.)
- **`annotations/`**: one UTF-8 plain-text transcription per page, with line breaks following the manuscript.
  - `part_01` is the **right** page and `part_02` is the **left** page of the scan with the same number.
  - A single `*` marks a word, or a run of words, that was illegible or missing because of damage, fading, or difficult calligraphy.

### `page-level/`: single-page images

Each scan has been split into its right and left pages (`*_right.png`, `*_left.png`).

**`page_level_text.csv`** (recommended): one row per page.

| column | description |
|---|---|
| `image_id` | `<page number>_part<N>`, e.g. `00000002_part1` |
| `image_path` | page image, relative to `page-level/`, e.g. `MANUSCRIPT_01/00000002_right.png` |
| `split` | `train`, `val`, or `test` |
| `book` | manuscript, e.g. `MANUSCRIPT_01` |
| `text` | full page transcription (may contain line breaks; read it with a CSV parser) |

`page_level.csv` has the same rows, but with an `annotation_path` column instead of `text`. Its paths are absolute paths from the original processing server, so use them only for their file names.


### `line-level/`: segmented line images

The line images were produced by the Start-Follow-Read (SFR) line-segmentation pipeline. Each one is a single straightened text line, named:

```
manuscript_<MS>_<page number>_part_<PP>_<line>.jpg
```

Each line was matched to its line of the page transcription.

- **`dataset.csv`**: all lines, with columns `img_path`, `text`, `split`.
- **`Manuscript NN/manuscript_NN.csv`**: lines for one manuscript, with columns `img_path`, `text`.

The directory prefix in `img_path` (`dataset/manuscript_NN/` or `images/`) does not match this repository's layout. Use the file name and look it up in `line-level/Manuscript NN/segmented_lines/`. 

```python
import pandas as pd
from pathlib import Path

root = Path("line-level")
df = pd.read_csv(root / "dataset.csv")
ms = df.img_path.str.extract(r"manuscript_(\d\d)_")[0]
df["file"] = [
    next((root / f"Manuscript {m}").glob("seg*_lines")) / Path(p).name
    for m, p in zip(ms, df.img_path)
]
```

## Splits

The data is split at the **page** level, so no page contributes lines to more than one split. Each line takes the split of the page it was cropped from.

| split | pages | lines |
|---|---:|---:|
| train | 1,545 | 26,328 |
| val | 196 | 3,395 |
| test | 194 | 3,351 |

## Manuscripts

| ID | Year (CE) | Script | Pages | Line images | Words | `*` markers |
|---|---|---|---:|---:|---:|---:|
| 01 | 1622 | Naskh | 404 | 2,895 | 39,842 | 490 |
| 02 | 1505 | Naskh | 26 | 465 | 5,994 | 44 |
| 03 | 1309 | Naskh | 14 | 187 | 2,503 | 1 |
| 04 | – | Ruq'ah | 18 | 193 | 1,686 | 5 |
| 05 | 1630 | Naskh | 126 | 2,901 | 40,291 | 890 |
| 06 | 1505 | Muhaqqaq | 42 | 776 | 12,169 | 98 |
| 07 | 1566 | Naskh | 30 | 498 | 7,066 | 0 |
| 08 | 1505 | Naskh | 40 | 449 | 3,726 | 0 |
| 09 | 979 | Ruq'ah | 172 | 1,556 | 12,388 | 2 |
| 10 | 1401 | Naskh | 36 | 667 | 10,792 | 50 |
| 11 | 1505 | Ruq'ah | 54 | 1,239 | 15,269 | 1 |
| 12 | 1176 | Naskh | 152 | 1,250 | 10,365 | 0 |
| 13 | 1497 | Ruq'ah | 22 | 295 | 3,321 | 18 |
| 14 | 1348 | Naskh | 40 | 637 | 7,590 | 31 |
| 15 | 1565 | Naskh | 212 | 3,787 | 48,035 | 121 |
| 16 | – | Naskh | 174 | 3,397 | 54,771 | 447 |
| 17 | 1198 | Ruq'ah | 26 | 433 | 4,928 | 22 |
| 18 | 1731 | Naskh | 22 | 262 | 3,057 | 0 |
| 19 | – | Ruq'ah | 266 | 5,417 | 39,303 | 138 |
| 20 | 1450 | Naskh | 340 | 5,770 | 90,971 | 271 |

## Baselines

| Model | Level | Test CER (raw) | Test WER (raw) | Test CER (norm.) | Test WER (norm.) |
|---|---|---:|---:|---:|---:|
| HATFormer (fine-tuned) | line | 13.91% | 34.79% | 12.26% | 30.14% |

"Normalized" scores are computed after canonicalising letter forms and removing optional diacritics. See the paper for confidence intervals, per-script results, and page-level results with Qwen3-VL.

## Acknowledgements

The source manuscripts are held by the Central Library of the Islamic University of Madinah. This dataset extends the collection effort of Najam and Faizullah (2024).

## Citation

```bibtex
@misc{faizullah2026khazain,
  title  = {Khazain Al-Hikma: A Cursive Arabic Handwriting Corpus from Islamic Heritage},
  author = {Faizullah, Safiullah and Attia, Maram and Selim, Amina and Zahran, Louai and
            Ashraf, Nouran and Tarek, Omnia and Ali, Rana and Said, Rawan and
            Yasser, Yomna and Khan, Imdad Ullah and Torki, Marwan},
  year   = {2026}
}
```

## License

_To be added._
