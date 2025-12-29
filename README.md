README.md

ECDIS Symbols Dataset (Chart No. 1) — Reproducible Extraction + Evaluation

Overview
This repo contains pre-extracted “row” artifacts from Chart No. 1 (pages 18–66). Each row is stored as:

* a row.json (structured metadata + text)
* a row.png (full row image)
* cell images (per column) including *_tight.png crops

Goal: let someone reproduce the experiment (view icons + run Gemini evaluation) without re-running the PDF extraction step.

What’s in the dataset
Expected folder layout (committed)
chart_rows_linked/
rows/
page_018/
row_001/
row.json
row.png
cells/
No.png
No_tight.png
INT.png
INT_tight.png
Description.png
Description_tight.png
NOAA.png
NOAA_tight.png
NGA.png
NGA_tight.png
OtherNGA.png
OtherNGA_tight.png
ECDIS.png
ECDIS_tight.png
page_019/
...
(optional generated outputs)
icon_samples.jsonl
icon_eval.csv

Row record format
Each row.json looks like this (example fields):

* id: unique row id like p018_r001
* page_number, row_index, no
* row_text: OCR/PDF-derived text for the row
* row_image: path to row.png
* cells: dict keyed by column name

  * each cell has:

    * text (best-effort extraction)
    * image / image_tight
    * bbox_pdf (PDF coordinates)
    * bbox_px_page (pixel coordinates at extraction DPI)

Quickstart (replicate experiment)

1. Clone the repo
   git clone <YOUR_REPO_URL>
   cd <YOUR_REPO_FOLDER>

2. Create a virtual environment
   python3 -m venv .venv
   source .venv/bin/activate

3. Install dependencies
   pip install -U pip
   pip install pandas matplotlib pillow opencv-python pytesseract python-dotenv google-genai streamlit

Notes:

* opencv-python provides cv2 (you do NOT pip install cv2)
* pytesseract requires the Tesseract binary installed on your OS

macOS (Tesseract install)
brew install tesseract

4. Verify the dataset paths
   From the repo root, you should be able to see:
   chart_rows_linked/rows/page_018/row_001/row.json
   chart_rows_linked/rows/page_018/row_001/cells/INT_tight.png

If your committed paths inside JSON reference “chart_rows/...”, keep the folder name consistent.
If you committed as chart_rows_linked/rows/ but the JSON paths start with chart_rows/, do one of the following:

Option A (recommended): create a symlink so paths resolve
ln -s chart_rows_linked/rows chart_rows

This creates:
chart_rows/  -> chart_rows_linked/rows/

Option B: rename the committed directory to match JSON paths
mv chart_rows_linked/rows chart_rows

Pick ONE. The rest of the scripts assume the image paths in row.json are valid.

Build the icon dataset (connect images to text)
This step creates icon_samples.jsonl: one record per icon-image cell, paired with the ground-truth label text (usually from Description).

Run:
python make_icon_dataset.py

Inputs:

* chart_rows_linked/rows (the extracted rows)

Outputs:

* chart_rows_linked/icon_samples.jsonl
* chart_rows_linked/icon_samples.summary.json

What it does:

* Iterates all rows
* For each row, pulls label_text from Description (fallback to row_text)
* For each icon column (INT/NOAA/NGA/OtherNGA/ECDIS), keeps the image if it is NOT blank and NOT text-only

View the icons + labels
Streamlit viewer (cycles and filter/search):
streamlit run viewer_app.py

You should point it to:
chart_rows_linked/icon_samples.jsonl

If your viewer expects a different filename, edit DEFAULT_JSONL in viewer_app.py.

Evaluate with Gemini (writes append-safe CSV)

1. Create a .env file in the repo root:
   GEMINI_API_KEY=your_key_here
   GEMINI_MODEL=gemini-2.5-flash

2. Run evaluation:
   python evaluate_all_to_csv.py

Outputs:

* chart_rows_linked/icon_eval.csv
  This file is appended after every icon so you can stop/restart without losing progress.

Generate human-readable charts
python simple_report.py /Users/jordantaylor/PycharmProjects/ECDIS_Symbols/chart_rows_linked/icon_eval.csv

Outputs (in the same folder as the CSV):

* A_scoreboard_correct_vs_wrong.png
* B_accuracy_by_column_with_counts.png
* C_confidence_correct_vs_wrong.png
  (and a short summary printed to console)

Reproduce your exact “286 icons” count
Your icon count depends on the prune rules in make_icon_dataset.py:

* blank threshold
* text-only detection thresholds
* which columns are included

To match your run:

* keep ICON_COLS the same
* do not change thresholds
* ensure your folder structure matches the paths in row.json

Troubleshooting

* “Missing image file” errors:
  Your JSON points to chart_rows/... but your repo folder is chart_rows_linked/rows/...
  Fix with symlink:
  ln -s chart_rows_linked/rows chart_rows

* OpenCV import error:
  pip install opencv-python

* pytesseract errors:
  Install the system tesseract binary (brew install tesseract) and ensure it’s on PATH.

* Streamlit shows missing images:
  Same as the path mismatch issue above; fix paths/symlink.

License / Source note
Chart No. 1 is published by NOAA/NGA/CHS. This repo contains derived image crops and metadata for research/benchmarking; ensure your use complies with the source’s terms and any applicable licensing.

