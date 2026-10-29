# Neural Style Transfer with AdaIN

A web app for applying the visual style of one image to the content of another. It uses Adaptive Instance Normalization (AdaIN), a VGG encoder, and a trained decoder, with a Flask interface for uploading images and adjusting style strength.

## Demo

The repository includes example input and output images in [`Demo_IO_Images`](Demo_IO_Images/) and a browser interface for trying your own images.

| Content | Style reference | Result |
| --- | --- | --- |
| ![Portrait content image](NST_Code/examples/brad_pitt.jpg) | ![Sketch style reference](NST_Code/examples/sketch.png) | ![Stylized portrait result](NST_Code/examples/stylized_brad_pitt.jpg) |

## Features

- Transfer a reference image's visual style onto a content image.
- Adjust the blend between original content and transferred style with the style-strength control.
- Preview uploads and download the generated result.
- Run inference on CUDA when available, or on CPU.
- Train the decoder with the included training script and your own image folders.

## Requirements

- Python 3.10–3.12
- pip
- About 2 GB of free disk space for the Python environment and model files
- Optional: a CUDA-compatible NVIDIA GPU for faster inference and training

The pretrained VGG encoder and decoder checkpoints are included under `NST_Code/`. They are large binary files, so make sure both `.pth` files are included when you upload or clone the repository.

## Run locally

From the repository root:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python NST_Code\app.py
```

On macOS or Linux, create and activate the environment with:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python NST_Code/app.py
```

Open [http://localhost:5000](http://localhost:5000), upload a content image and a style image (`.jpg`, `.jpeg`, or `.png`), set the style strength, and submit. The first start loads both model checkpoints and can take a little while, especially on CPU.

### Configuration

Set a unique Flask secret key before running outside a local development environment. PowerShell:

```powershell
$env:SECRET_KEY = "replace-with-a-long-random-value"
python NST_Code\app.py
```

Bash:

```bash
export SECRET_KEY="replace-with-a-long-random-value"
python NST_Code/app.py
```

## Train the decoder

`NST_Code/train.py` defaults to the example content and style image folders included in `NST_Code/`. To train on your own images, pass folders with image files directly inside them:

```powershell
python NST_Code\train.py `
  --content_dir "C:\path\to\content_images" `
  --style_dir "C:\path\to\style_images" `
  --vgg NST_Code\vgg_normalised.pth `
  --experiment my_experiment `
  --epochs 200
```

Images should be directly inside each directory. Training checkpoints and sample outputs are written under `NST_Code/experiment/<experiment-name>/`. The web app uses the included checkpoint at `NST_Code/experiment/final_exp/decoder_final.pth`; update `decoder_path` in `NST_Code/app.py` if you want to serve a different trained decoder.

## Project structure

```text
NST_Code/
├── app.py                         # Flask app and inference pipeline
├── train.py                       # Decoder training script
├── vgg_normalised.pth             # VGG encoder weights
├── experiment/final_exp/
│   └── decoder_final.pth          # Decoder weights used by the app
├── templates/index.html           # Web interface
├── examples/                      # Gallery examples
└── utils/                          # Model definitions and AdaIN utilities
Demo_IO_Images/                     # Additional example inputs and outputs
requirements.txt
```

## Model notes

Inference encodes the content and style images, applies AdaIN to align feature statistics, blends those features using the selected strength, then decodes the result. The encoder weights and decoder checkpoint are loaded from paths relative to `NST_Code/`, so the app can be launched from the repository root or from that directory.

## Known limitations

- Inference runs synchronously in the Flask request, so large images and CPU-only machines can take longer to process.
- Uploaded images are stored in `NST_Code/static/uploads/` for the local app; generated and uploaded files are ignored by Git.
- This repository is a local demo. A public deployment should add production hosting, upload limits, and appropriate user-data retention controls.

## License

No license file is included yet. Add a license before inviting others to reuse or redistribute the project.
