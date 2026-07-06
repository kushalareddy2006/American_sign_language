# SIGN4ALL - Complete Source Code

## config\config.yaml
``yaml
project:
  name: SIGN4ALL
  seed: 42

paths:
  dataset_dir: dataset/Classification_Dataset
  outputs_dir: outputs
  saved_models_dir: saved_models
  class_indices: outputs/class_indices.json

data:
  image_size: [224, 224]
  batch_size: 32
  validation_split: 0.15
  test_split: 0.15
  classes: ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z", "SPACE"]
  augmentation:
    rotation_range: 15
    zoom_range: 0.15
    width_shift_range: 0.1
    height_shift_range: 0.1
    brightness_range: [0.8, 1.2]

training:
  epochs: 30
  fine_tune_epochs: 10
  learning_rate: 0.001
  fine_tune_learning_rate: 0.00001
  mixed_precision: true
  patience: 6
  gradient_clipnorm: 1.0

inference:
  confidence_threshold: 0.65
  smoothing_window: 8
  stable_frames: 5
  cooldown_seconds: 0.7
  camera_id: 0
````

## docs_MODEL_COMPARISON.md
``markdown
# SIGN4ALL Model Comparison Report

This report is populated by running evaluation scripts. JSON reports are saved in `outputs/reports/`.

| Model | Input | Strength | Best use |
| --- | --- | --- | --- |
| MobileNetV2 | RGB image | Fast transfer learning baseline | CPU/GPU image classifier |
| EfficientNetB0 | RGB image | Strong accuracy/parameter tradeoff | High-quality CNN baseline |
| DenseNet121 | RGB image | Feature reuse with dense connections | Research comparison |
| ResNet18/34 | RGB image | Custom residual baseline | Skip-connection analysis |
| MediaPipe + MLP | 63 landmarks | Low latency and robust real-time use | Primary webcam model |
| Hybrid Fusion | RGB + landmarks | Combines visual and geometric cues | Best research implementation |

Recommended production path:

1. Train `mediapipe_mlp` first for real-time inference.
2. Train `mobilenetv2` as a visual baseline.
3. Train `hybrid` once landmark extraction quality is good.
4. Compare `accuracy`, `precision`, `recall`, `f1_score`, and `top3_accuracy` in the dashboard.
````

## README.md
``markdown
# SIGN4ALL

SIGN4ALL is a Windows/VS Code-ready real-time ASL alphabet recognition system using Python, TensorFlow/Keras, OpenCV, MediaPipe, and Scikit-learn.

It supports 27 classes: `A-Z` plus `SPACE`.

## Project Structure

```text
SIGN4ALL/
|-- config/config.yaml
|-- dataset/
|-- models/
|-- notebooks/
|-- outputs/
|   |-- plots/
|   |-- confusion_matrix/
|   |-- gradcam/
|   `-- logs/
|-- saved_models/
|-- scripts/
|-- src/
|   |-- preprocessing/
|   |-- training/
|   |-- evaluation/
|   |-- inference/
|   |-- visualization/
|   `-- utils/
`-- requirements.txt
```

## Setup

```powershell
cd C:\Users\kusha\Downloads\Deep_learning-Project\SIGN4ALL
python -m venv .venv
.\.venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If activation is blocked, run commands through:

```powershell
.\.venv\Scripts\python.exe
```

## Dataset

If your class folders are directly inside `dataset`, keep this in `config/config.yaml`:

```yaml
dataset_dir: dataset
```

Expected layout:

```text
dataset/
|-- a/
|-- b/
|-- ...
|-- z/
`-- space/
```

Analyze dataset and create clean splits:

```powershell
python scripts\analyze_dataset.py
```

## Train Models

Primary real-time model:

```powershell
python scripts\train_model.py --model mediapipe_mlp
```

CNN and hybrid models:

```powershell
python scripts\train_model.py --model mobilenetv2
python scripts\train_model.py --model efficientnetb0
python scripts\train_model.py --model densenet121
python scripts\train_model.py --model resnet18
python scripts\train_model.py --model resnet34
python scripts\train_model.py --model hybrid
```

Train every model sequentially:

```powershell
python scripts\train_all_models.py
```

## Evaluate Models

MediaPipe MLP:

```powershell
python scripts\evaluate_model.py --model-path saved_models\mediapipe_mlp.keras --model-type mediapipe_mlp
```

Image models:

```powershell
python scripts\evaluate_model.py --model-path saved_models\mobilenetv2.keras --model-type image
python scripts\evaluate_model.py --model-path saved_models\efficientnetb0.keras --model-type image
python scripts\evaluate_model.py --model-path saved_models\densenet121.keras --model-type image
python scripts\evaluate_model.py --model-path saved_models\resnet18.keras --model-type image
python scripts\evaluate_model.py --model-path saved_models\resnet34.keras --model-type image
```

Hybrid:

```powershell
python scripts\evaluate_model.py --model-path saved_models\hybrid.keras --model-type hybrid
```

Evaluation saves:

- `outputs/reports/`
- `outputs/confusion_matrix/`
- `outputs/predictions/`
- `outputs/plots/`

## Generate Visualizations

```powershell
python scripts\generate_model_visualizations.py --model-path saved_models\mediapipe_mlp.keras --model-type mediapipe_mlp
python scripts\generate_model_visualizations.py --model-path saved_models\mobilenetv2.keras --model-type mobilenetv2
python scripts\generate_model_visualizations.py --model-path saved_models\efficientnetb0.keras --model-type efficientnetb0
python scripts\generate_model_visualizations.py --model-path saved_models\densenet121.keras --model-type densenet121
python scripts\generate_model_visualizations.py --model-path saved_models\resnet18.keras --model-type resnet18
python scripts\generate_model_visualizations.py --model-path saved_models\resnet34.keras --model-type resnet34
python scripts\generate_model_visualizations.py --model-path saved_models\hybrid.keras --model-type hybrid
```

## Real-Time Webcam

Run the real-time OpenCV interface:

```powershell
python scripts\realtime_webcam.py --model-path saved_models\mediapipe_mlp.keras --model-type mediapipe_mlp
```

Controls:

- `q`: quit webcam
- `c`: clear sentence
- `v`: speak sentence when `--speak` is enabled

With voice:

```powershell
python scripts\realtime_webcam.py --model-path saved_models\mediapipe_mlp.keras --model-type mediapipe_mlp --speak
```

## After Laptop Restart

You do not need to retrain. Run only:

```powershell
cd C:\Users\kusha\Downloads\Deep_learning-Project\SIGN4ALL
.\.venv\Scripts\activate
python scripts\realtime_webcam.py --model-path saved_models\mediapipe_mlp.keras --model-type mediapipe_mlp
```

## Notes

- Do not use horizontal flipping for ASL letters.
- If webcam index `0` fails, try `--camera 1`.
- The TensorFlow oneDNN messages are normal and can be ignored.
- Use `mediapipe_mlp.keras` for the fastest real-time recognition.
````

## requirements.txt
``text
tensorflow>=2.15,<2.17
opencv-python>=4.9
mediapipe==0.10.14
scikit-learn>=1.4
numpy==1.26.4
ml-dtypes==0.3.2
pandas>=2.0
matplotlib>=3.8
seaborn>=0.13
Pillow>=10.0
PyYAML>=6.0
tqdm>=4.66
plotly>=5.19
pyttsx3>=2.90
joblib>=1.3
````

## scripts\analyze_dataset.py
``python
from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

from src.preprocessing.dataset import remove_corrupted_images, scan_dataset, split_dataframe
from src.utils.config import ROOT, load_config
from src.visualization.plots import plot_class_distribution, plot_sample_images


def main() -> None:
    config = load_config()
    outputs = ROOT / config["paths"]["outputs_dir"]
    outputs.mkdir(exist_ok=True)
    df, corrupted = remove_corrupted_images(scan_dataset(ROOT / config["paths"]["dataset_dir"]))
    df.to_csv(outputs / "dataset_inventory.csv", index=False)
    train_df, val_df, test_df = split_dataframe(
        df,
        config["data"]["validation_split"],
        config["data"]["test_split"],
        config["project"]["seed"],
    )
    train_df.to_csv(outputs / "train_split.csv", index=False)
    val_df.to_csv(outputs / "val_split.csv", index=False)
    test_df.to_csv(outputs / "test_split.csv", index=False)
    Path(outputs / "corrupted_images.txt").write_text("\n".join(corrupted), encoding="utf-8")
    plot_class_distribution(df, outputs / "plots" / "dataset_class_distribution.png")
    plot_sample_images(df, outputs / "plots" / "sample_images.png")
    print(
        f"Images: {len(df)} | Train: {len(train_df)} | "
        f"Validation: {len(val_df)} | Test: {len(test_df)} | Corrupted: {len(corrupted)}"
    )


if __name__ == "__main__":
    main()
````

## scripts\evaluate_model.py
``python
from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

from src.evaluation.evaluate import main


if __name__ == "__main__":
    main()
````

## scripts\generate_model_visualizations.py
``python
from __future__ import annotations

import argparse
from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

import pandas as pd
import tensorflow as tf

from src.preprocessing.dataset import class_indices, remove_corrupted_images, scan_dataset, split_dataframe
from src.utils.config import ROOT, load_config
from src.visualization.model_visualizer import (
    save_augmented_grid,
    save_cnn_visualizations,
    save_landmark_visualization,
    save_prediction_grid,
)
from src.visualization.plots import plot_class_distribution, plot_sample_images


CNN_MODELS = {"mobilenetv2", "efficientnetb0", "densenet121", "resnet18", "resnet34"}


def main() -> None:
    parser = argparse.ArgumentParser(description="Generate SIGN4ALL visualizations for one trained model")
    parser.add_argument("--model-path", required=True)
    parser.add_argument("--model-type", required=True, choices=["mobilenetv2", "efficientnetb0", "densenet121", "resnet18", "resnet34", "mediapipe_mlp", "hybrid"])
    parser.add_argument("--config", default="config/config.yaml")
    args = parser.parse_args()

    config = load_config(args.config)
    outputs_dir = ROOT / config["paths"]["outputs_dir"]
    model_name = Path(args.model_path).stem
    model_plot_dir = outputs_dir / "plots" / model_name
    model_plot_dir.mkdir(parents=True, exist_ok=True)

    df, _ = remove_corrupted_images(scan_dataset(ROOT / config["paths"]["dataset_dir"]))
    _, _, test_df = split_dataframe(df, config["data"]["validation_split"], config["data"]["test_split"], config["project"]["seed"])
    classes = list(class_indices(config["data"]["classes"]).keys())
    sample_path = test_df.iloc[0]["path"]

    plot_class_distribution(df, model_plot_dir / "dataset_class_distribution.png")
    plot_sample_images(df, model_plot_dir / "sample_images.png")
    save_augmented_grid(sample_path, model_plot_dir / "augmented_images.png")
    save_landmark_visualization(sample_path, model_plot_dir / "landmark_skeleton.png")

    model = tf.keras.models.load_model(args.model_path)
    save_prediction_grid(model, test_df, classes, model_plot_dir / "prediction_visualizations.png", args.model_type)
    if args.model_type in CNN_MODELS:
        save_cnn_visualizations(model, sample_path, outputs_dir / "gradcam", model_name)

    print(f"Saved visualizations to {model_plot_dir}")


if __name__ == "__main__":
    main()
````

## scripts\prepare_dataset.py
``python
from __future__ import annotations

import argparse
import shutil
import zipfile
from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

from src.utils.config import ROOT


def main() -> None:
    parser = argparse.ArgumentParser(description="Prepare SIGN4ALL dataset from a zip archive or folder")
    parser.add_argument("--source", required=True, help="Path to archive.zip or an extracted Classification_Dataset folder")
    parser.add_argument("--target", default=str(ROOT / "dataset"))
    args = parser.parse_args()
    source = Path(args.source)
    target = Path(args.target)
    target.mkdir(parents=True, exist_ok=True)

    if source.suffix.lower() == ".zip":
        with zipfile.ZipFile(source, "r") as zip_ref:
            zip_ref.extractall(target)
        print(f"Extracted {source} to {target}")
    elif source.is_dir():
        destination = target / source.name
        if destination.exists():
            shutil.rmtree(destination)
        shutil.copytree(source, destination)
        print(f"Copied {source} to {destination}")
    else:
        raise FileNotFoundError(source)


if __name__ == "__main__":
    main()
````

## scripts\realtime_webcam.py
``python
from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

from src.inference.realtime import main


if __name__ == "__main__":
    main()
````

## scripts\train_all_models.py
``python
from __future__ import annotations

from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

from src.training.train import train_hybrid_model, train_image_model, train_landmark_model
from src.utils.config import load_config


def main() -> None:
    config = load_config()
    for model_name in ["mobilenetv2", "efficientnetb0", "densenet121", "resnet18", "resnet34"]:
        train_image_model(model_name, config)
    train_landmark_model(config)
    train_hybrid_model(config)


if __name__ == "__main__":
    main()
````

## scripts\train_model.py
``python
from pathlib import Path
import sys

PROJECT_ROOT = Path(__file__).resolve().parents[1]
if str(PROJECT_ROOT) not in sys.path:
    sys.path.insert(0, str(PROJECT_ROOT))

from src.training.train import main


if __name__ == "__main__":
    main()
````

## src\__init__.py
``python
"""SIGN4ALL source package."""
````

## src\evaluation\__init__.py
``python
"""Evaluation utilities."""
````

## src\evaluation\evaluate.py
``python
from __future__ import annotations

import argparse
import json
from pathlib import Path

import numpy as np
import pandas as pd
import tensorflow as tf
from sklearn.metrics import accuracy_score, classification_report, precision_recall_fscore_support, top_k_accuracy_score

from src.preprocessing.dataset import class_indices, dataframe_to_dataset, scan_dataset, split_dataframe
from src.preprocessing.landmarks import build_landmark_table
from src.training.train import HybridSequence
from src.utils.config import ROOT, load_config
from src.visualization.plots import plot_confusion, plot_misclassified, plot_roc_pr, plot_tsne


def prepare_test_split(config: dict) -> tuple[pd.DataFrame, dict[str, int], list[str]]:
    outputs_dir = ROOT / config["paths"]["outputs_dir"]
    indices = class_indices(config["data"]["classes"])
    classes = list(indices.keys())
    test_csv = outputs_dir / "test_split.csv"
    if test_csv.exists():
        test_df = pd.read_csv(test_csv)
    else:
        _, _, test_df = split_dataframe(
            scan_dataset(ROOT / config["paths"]["dataset_dir"]),
            config["data"]["validation_split"],
            config["data"]["test_split"],
            config["project"]["seed"],
        )
        outputs_dir.mkdir(parents=True, exist_ok=True)
        test_df.to_csv(test_csv, index=False)
    return test_df, indices, classes


def save_report(
    model_path: str | Path,
    y_true: np.ndarray,
    y_pred: np.ndarray,
    probabilities: np.ndarray,
    classes: list[str],
    image_paths: list[str],
    outputs_dir: Path,
) -> dict:
    precision, recall, f1, _ = precision_recall_fscore_support(y_true, y_pred, average="weighted", zero_division=0)
    report = {
        "accuracy": float(accuracy_score(y_true, y_pred)),
        "precision": float(precision),
        "recall": float(recall),
        "f1_score": float(f1),
        "top3_accuracy": float(top_k_accuracy_score(y_true, probabilities, k=3, labels=list(range(len(classes))))),
        "classification_report": classification_report(y_true, y_pred, target_names=classes, zero_division=0),
    }
    model_name = Path(model_path).stem
    report_dir = outputs_dir / "reports"
    predictions_dir = outputs_dir / "predictions"
    report_dir.mkdir(parents=True, exist_ok=True)
    predictions_dir.mkdir(parents=True, exist_ok=True)
    (report_dir / f"{model_name}_report.json").write_text(json.dumps(report, indent=2), encoding="utf-8")
    pd.DataFrame(probabilities, columns=classes).assign(
        true=[classes[i] for i in y_true],
        pred=[classes[i] for i in y_pred],
    ).to_csv(predictions_dir / f"{model_name}_predictions.csv", index=False)
    plot_confusion(y_true, y_pred, classes, outputs_dir / "confusion_matrix" / f"{model_name}.png")
    plot_roc_pr(y_true, probabilities, classes, outputs_dir / "plots" / model_name)
    plot_misclassified(
        image_paths,
        [classes[i] for i in y_true],
        [classes[i] for i in y_pred],
        outputs_dir / "plots" / f"{model_name}_misclassified.png",
    )
    plot_tsne(probabilities, y_true, classes, outputs_dir / "plots" / model_name / "probability_tsne.png")
    return report


def evaluate_image_model(model_path: str | Path, config: dict) -> dict:
    outputs_dir = ROOT / config["paths"]["outputs_dir"]
    test_df, indices, classes = prepare_test_split(config)
    dataset = dataframe_to_dataset(test_df, indices, tuple(config["data"]["image_size"]), config["data"]["batch_size"], training=False)
    model = tf.keras.models.load_model(model_path)
    probabilities = model.predict(dataset)
    y_true = test_df["label"].map(indices).values
    y_pred = probabilities.argmax(axis=1)
    return save_report(model_path, y_true, y_pred, probabilities, classes, test_df["path"].tolist(), outputs_dir)


def evaluate_landmark_model(model_path: str | Path, config: dict) -> dict:
    outputs_dir = ROOT / config["paths"]["outputs_dir"]
    test_df, indices, classes = prepare_test_split(config)
    table = build_landmark_table(test_df, outputs_dir / "landmarks_test.csv")
    x_test = table[[f"f{i}" for i in range(63)]].values.astype(np.float32)
    y_true = table["label"].map(indices).values.astype(np.int32)
    model = tf.keras.models.load_model(model_path)
    probabilities = model.predict(x_test)
    y_pred = probabilities.argmax(axis=1)
    return save_report(model_path, y_true, y_pred, probabilities, classes, table["path"].tolist(), outputs_dir)


def evaluate_hybrid_model(model_path: str | Path, config: dict) -> dict:
    outputs_dir = ROOT / config["paths"]["outputs_dir"]
    test_df, indices, classes = prepare_test_split(config)
    table = build_landmark_table(test_df, outputs_dir / "hybrid_landmarks_test.csv")
    sequence = HybridSequence(table, indices, config["data"]["batch_size"], tuple(config["data"]["image_size"]), shuffle=False)
    model = tf.keras.models.load_model(model_path)
    probabilities = model.predict(sequence)
    y_true = table["label"].map(indices).values.astype(np.int32)
    y_pred = probabilities.argmax(axis=1)
    return save_report(model_path, y_true, y_pred, probabilities, classes, table["path"].tolist(), outputs_dir)


def main() -> None:
    parser = argparse.ArgumentParser(description="Evaluate a trained SIGN4ALL model")
    parser.add_argument("--model-path", required=True)
    parser.add_argument("--model-type", default="image", choices=["image", "mediapipe_mlp", "hybrid"])
    parser.add_argument("--config", default="config/config.yaml")
    args = parser.parse_args()
    config = load_config(args.config)
    if args.model_type == "mediapipe_mlp":
        report = evaluate_landmark_model(args.model_path, config)
    elif args.model_type == "hybrid":
        report = evaluate_hybrid_model(args.model_path, config)
    else:
        report = evaluate_image_model(args.model_path, config)
    print(json.dumps({k: v for k, v in report.items() if k != "classification_report"}, indent=2))


if __name__ == "__main__":
    main()
````

## src\inference\__init__.py
``python
"""Inference helpers."""
````

## src\inference\ensemble.py
``python
from __future__ import annotations

from collections import Counter
from pathlib import Path

import numpy as np

from src.inference.predictor import SignPredictor
from src.utils.config import ROOT


MODEL_TYPE_BY_NAME = {
    "mediapipe_mlp": "mediapipe_mlp",
    "mobilenetv2": "mobilenetv2",
    "efficientnetb0": "efficientnetb0",
    "densenet121": "densenet121",
    "resnet18": "resnet18",
    "resnet34": "resnet34",
    "hybrid": "hybrid",
}


def infer_model_type(model_path: str | Path) -> str | None:
    name = Path(model_path).stem.lower()
    for key, value in MODEL_TYPE_BY_NAME.items():
        if key in name:
            return value
    return None


def discover_supported_models(saved_dir: str | Path | None = None) -> list[tuple[Path, str]]:
    saved_dir = Path(saved_dir) if saved_dir else ROOT / "saved_models"
    models: list[tuple[Path, str]] = []
    for path in sorted(saved_dir.glob("*.keras")) + sorted(saved_dir.glob("*.h5")):
        model_type = infer_model_type(path)
        if model_type:
            models.append((path, model_type))
    return models


class EnsembleSignPredictor:
    """Majority-vote predictor across all available saved SIGN4ALL models."""

    def __init__(self, model_specs: list[tuple[Path, str]], confidence_threshold: float = 0.50):
        if not model_specs:
            raise FileNotFoundError("No supported saved models were found for ensemble prediction.")
        self.predictors = [
            SignPredictor(path, model_type=model_type, confidence_threshold=confidence_threshold)
            for path, model_type in model_specs
        ]
        self.confidence_threshold = confidence_threshold
        self.classes = self.predictors[0].classes

    def close(self) -> None:
        for predictor in self.predictors:
            predictor.close()

    def predict_frame(self, frame_bgr: np.ndarray) -> dict:
        raw_results = []
        vote_labels = []
        confidences = []
        bbox = None

        for predictor in self.predictors:
            try:
                result = predictor.predict_frame(frame_bgr)
            except Exception:
                continue
            raw_results.append(result)
            if result.get("bbox") is not None:
                bbox = result["bbox"]
            label = result.get("label", "")
            confidence = float(result.get("confidence", 0.0))
            if label and confidence >= self.confidence_threshold:
                vote_labels.append(label)
                confidences.append(confidence)

        if vote_labels:
            label, votes = Counter(vote_labels).most_common(1)[0]
            matching_conf = [conf for vote, conf in zip(vote_labels, confidences) if vote == label]
            confidence = float(np.mean(matching_conf)) if matching_conf else 0.0
        elif raw_results:
            best = max(raw_results, key=lambda item: item.get("confidence", 0.0))
            label = best.get("label", "")
            confidence = float(best.get("confidence", 0.0))
            votes = 1 if label else 0
        else:
            label, confidence, votes = "", 0.0, 0

        return {
            "label": label,
            "confidence": confidence,
            "bbox": bbox,
            "votes": votes,
            "model_count": len(self.predictors),
            "raw_results": raw_results,
            "probabilities": None,
        }
````

## src\inference\predictor.py
``python
from __future__ import annotations

import json
from pathlib import Path

import cv2
import numpy as np
import tensorflow as tf

from src.preprocessing.landmarks import HandLandmarkExtractor
from src.utils.config import ROOT
from src.utils.constants import CLASSES


class SignPredictor:
    def __init__(
        self,
        model_path: str | Path,
        model_type: str = "mediapipe_mlp",
        class_indices_path: str | Path | None = None,
        confidence_threshold: float = 0.65,
    ):
        self.model = tf.keras.models.load_model(model_path)
        self.model_type = model_type
        self.confidence_threshold = confidence_threshold
        self.extractor = HandLandmarkExtractor(static_image_mode=False)
        self.classes = self._load_classes(class_indices_path)

    def _load_classes(self, path: str | Path | None) -> list[str]:
        if path and Path(path).exists():
            data = json.loads(Path(path).read_text(encoding="utf-8"))
            return [item[0] for item in sorted(data.items(), key=lambda item: item[1])]
        default_path = ROOT / "outputs/class_indices.json"
        if default_path.exists():
            data = json.loads(default_path.read_text(encoding="utf-8"))
            return [item[0] for item in sorted(data.items(), key=lambda item: item[1])]
        return CLASSES

    def close(self) -> None:
        self.extractor.close()

    def predict_frame(self, frame_bgr: np.ndarray) -> dict:
        if self.model_type in {"mediapipe_mlp", "landmark_mlp", "mlp"}:
            result = self.extractor.extract(frame_bgr)
            if result.vector is None:
                return {"label": "", "confidence": 0.0, "bbox": None, "probabilities": None}
            probabilities = self.model.predict(result.vector.reshape(1, -1), verbose=0)[0]
            idx = int(np.argmax(probabilities))
            return {
                "label": self.classes[idx],
                "confidence": float(probabilities[idx]),
                "bbox": result.bbox,
                "probabilities": probabilities,
            }

        if self.model_type == "hybrid":
            result = self.extractor.extract(frame_bgr)
            if result.vector is None:
                return {"label": "", "confidence": 0.0, "bbox": None, "probabilities": None}
            image = cv2.cvtColor(frame_bgr, cv2.COLOR_BGR2RGB)
            image = cv2.resize(image, (224, 224)).astype(np.float32) / 255.0
            probabilities = self.model.predict(
                {"image": image[None, ...], "landmarks": result.vector.reshape(1, -1)},
                verbose=0,
            )[0]
            idx = int(np.argmax(probabilities))
            return {
                "label": self.classes[idx],
                "confidence": float(probabilities[idx]),
                "bbox": result.bbox,
                "probabilities": probabilities,
            }

        image = cv2.cvtColor(frame_bgr, cv2.COLOR_BGR2RGB)
        image = cv2.resize(image, (224, 224)).astype(np.float32) / 255.0
        probabilities = self.model.predict(image[None, ...], verbose=0)[0]
        idx = int(np.argmax(probabilities))
        return {"label": self.classes[idx], "confidence": float(probabilities[idx]), "bbox": None, "probabilities": probabilities}


def draw_overlay(frame: np.ndarray, label: str, confidence: float, fps: float, bbox: tuple[int, int, int, int] | None = None) -> np.ndarray:
    if bbox:
        x1, y1, x2, y2 = bbox
        cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 220, 80), 2)
    text = f"{label}: {confidence:.2f}" if label else "No confident sign"
    cv2.putText(frame, text, (20, 38), cv2.FONT_HERSHEY_SIMPLEX, 1, (30, 240, 80), 2)
    cv2.putText(frame, f"FPS: {fps:.1f}", (20, 76), cv2.FONT_HERSHEY_SIMPLEX, 0.8, (255, 220, 50), 2)
    return frame
````

## src\inference\realtime.py
``python
from __future__ import annotations

import argparse
import time
from pathlib import Path

import cv2
import pyttsx3

from src.inference.predictor import SignPredictor, draw_overlay
from src.inference.sentence import PredictionSmoother, SentenceBuilder, suggest_words
from src.utils.config import ROOT, load_config


def run_webcam(model_path: str | Path, model_type: str, camera_id: int, config: dict, speak: bool = False) -> None:
    threshold = config["inference"]["confidence_threshold"]
    smoother = PredictionSmoother(config["inference"]["smoothing_window"], threshold)
    sentence = SentenceBuilder(config["inference"]["stable_frames"], config["inference"]["cooldown_seconds"])
    predictor = SignPredictor(model_path, model_type=model_type, confidence_threshold=threshold)
    engine = pyttsx3.init() if speak else None
    cap = cv2.VideoCapture(camera_id)
    previous = time.time()
    try:
        while cap.isOpened():
            ok, frame = cap.read()
            if not ok:
                break
            frame = cv2.flip(frame, 1)
            result = predictor.predict_frame(frame)
            label, confidence = smoother.update(result["label"], result["confidence"])
            text = sentence.update(label)
            now = time.time()
            fps = 1.0 / max(now - previous, 1e-6)
            previous = now
            frame = predictor.extractor.draw(frame)
            frame = draw_overlay(frame, label, confidence, fps, result["bbox"])
            cv2.putText(frame, text[-45:], (20, frame.shape[0] - 50), cv2.FONT_HERSHEY_SIMPLEX, 0.9, (255, 255, 255), 2)
            cv2.putText(frame, "Suggestions: " + ", ".join(suggest_words(text)), (20, frame.shape[0] - 18), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (200, 220, 255), 1)
            cv2.imshow("SIGN4ALL Real-Time ASL Recognition", frame)
            key = cv2.waitKey(1) & 0xFF
            if key == ord("q"):
                break
            if key == ord("c"):
                sentence.clear()
            if key == ord("v") and engine:
                engine.say(text)
                engine.runAndWait()
    finally:
        cap.release()
        predictor.close()
        cv2.destroyAllWindows()


def main() -> None:
    parser = argparse.ArgumentParser(description="Run SIGN4ALL webcam inference")
    parser.add_argument("--model-path", default=str(ROOT / "saved_models/mediapipe_mlp.keras"))
    parser.add_argument("--model-type", default="mediapipe_mlp")
    parser.add_argument("--camera", type=int, default=0)
    parser.add_argument("--speak", action="store_true")
    args = parser.parse_args()
    config = load_config()
    run_webcam(args.model_path, args.model_type, args.camera, config, speak=args.speak)


if __name__ == "__main__":
    main()
````

## src\inference\sentence.py
``python
from __future__ import annotations

import time
from collections import Counter, deque


class PredictionSmoother:
    def __init__(self, window: int = 8, threshold: float = 0.65):
        self.window = deque(maxlen=window)
        self.threshold = threshold

    def update(self, label: str, confidence: float) -> tuple[str, float]:
        if confidence >= self.threshold:
            self.window.append((label, confidence))
        if not self.window:
            return "", 0.0
        labels = [item[0] for item in self.window]
        label = Counter(labels).most_common(1)[0][0]
        conf = sum(c for l, c in self.window if l == label) / max(labels.count(label), 1)
        return label, conf


class SentenceBuilder:
    def __init__(self, stable_frames: int = 5, cooldown_seconds: float = 0.7):
        self.text = ""
        self.last_label = ""
        self.pending_label = ""
        self.pending_count = 0
        self.last_commit_time = 0.0
        self.stable_frames = stable_frames
        self.cooldown_seconds = cooldown_seconds

    def update(self, label: str) -> str:
        if not label:
            return self.text
        if label == self.pending_label:
            self.pending_count += 1
        else:
            self.pending_label = label
            self.pending_count = 1

        now = time.time()
        if self.pending_count >= self.stable_frames and label != self.last_label and now - self.last_commit_time > self.cooldown_seconds:
            self.text += " " if label == "SPACE" else label
            self.last_label = label
            self.last_commit_time = now
        return self.text

    def clear(self) -> None:
        self.text = ""
        self.last_label = ""
        self.pending_label = ""
        self.pending_count = 0


def suggest_words(text: str, vocabulary: list[str] | None = None, limit: int = 5) -> list[str]:
    vocabulary = vocabulary or [
        "HELLO", "YES", "NO", "THANK", "PLEASE", "HELP", "LOVE", "FAMILY", "FRIEND",
        "SCHOOL", "WORK", "TECHNOLOGY", "ABOUT", "ABLE", "SIGN", "LANGUAGE",
    ]
    prefix = text.strip().split(" ")[-1].upper() if text.strip() else ""
    if not prefix:
        return []
    return [word for word in vocabulary if word.startswith(prefix)][:limit]
````

## src\models\__init__.py
``python
"""Model builders for SIGN4ALL."""
````

## src\models\builders.py
``python
from __future__ import annotations

import tensorflow as tf

from src.models.densenet121_model import build_densenet121_model
from src.models.efficientnetb0_model import build_efficientnetb0_model
from src.models.hybrid_model import build_hybrid_model
from src.models.mediapipe_mlp_model import build_mediapipe_mlp_model
from src.models.mobilenetv2_model import build_mobilenetv2_model
from src.models.resnet_model import build_resnet_model


def compile_model(model: tf.keras.Model, lr: float, clipnorm: float = 1.0) -> tf.keras.Model:
    optimizer = tf.keras.optimizers.Adam(learning_rate=lr, clipnorm=clipnorm)
    model.compile(
        optimizer=optimizer,
        loss="sparse_categorical_crossentropy",
        metrics=["accuracy", tf.keras.metrics.SparseTopKCategoricalAccuracy(k=3, name="top3_accuracy")],
    )
    return model


def build_model(name: str, num_classes: int = 27) -> tf.keras.Model:
    name = name.lower()
    if name == "mobilenetv2":
        return build_mobilenetv2_model(num_classes=num_classes)
    if name == "efficientnetb0":
        return build_efficientnetb0_model(num_classes=num_classes)
    if name == "densenet121":
        return build_densenet121_model(num_classes=num_classes)
    if name in {"resnet18", "resnet34"}:
        return build_resnet_model(version=name, num_classes=num_classes)
    if name in {"mediapipe_mlp", "landmark_mlp", "mlp"}:
        return build_mediapipe_mlp_model(num_classes=num_classes)
    if name == "hybrid":
        return build_hybrid_model(num_classes=num_classes)
    raise ValueError(f"Unknown model name: {name}")
````

## src\models\densenet121_model.py
``python
from __future__ import annotations

import tensorflow as tf
from tensorflow.keras import layers, regularizers


def build_densenet121_model(
    num_classes: int = 27,
    input_shape: tuple[int, int, int] = (224, 224, 3),
    trainable_backbone: bool = False,
) -> tf.keras.Model:
    """Build DenseNet121 transfer-learning ASL classifier."""
    base_model = tf.keras.applications.DenseNet121(
        include_top=False,
        weights="imagenet",
        input_shape=input_shape,
    )
    base_model.trainable = trainable_backbone

    inputs = layers.Input(shape=input_shape, name="image")
    x = tf.keras.applications.densenet.preprocess_input(inputs * 255.0)
    x = base_model(x, training=False)
    x = layers.GlobalAveragePooling2D(name="global_average_pooling")(x)
    x = layers.BatchNormalization(name="batch_norm")(x)
    x = layers.Dropout(0.35, name="dropout_035")(x)
    x = layers.Dense(
        192,
        activation="relu",
        kernel_regularizer=regularizers.l2(0.001),
        name="dense_192_relu_l2",
    )(x)
    x = layers.Dropout(0.25, name="dropout_025")(x)
    outputs = layers.Dense(num_classes, activation="softmax", dtype="float32", name="asl_softmax")(x)
    model = tf.keras.Model(inputs, outputs, name="sign4all_densenet121")
    model.backbone = base_model
    return model
````

## src\models\efficientnetb0_model.py
``python
from __future__ import annotations

import tensorflow as tf
from tensorflow.keras import layers, regularizers


def build_efficientnetb0_model(
    num_classes: int = 27,
    input_shape: tuple[int, int, int] = (224, 224, 3),
    trainable_backbone: bool = False,
) -> tf.keras.Model:
    """Build EfficientNetB0 transfer-learning ASL classifier."""
    base_model = tf.keras.applications.EfficientNetB0(
        include_top=False,
        weights="imagenet",
        input_shape=input_shape,
    )
    base_model.trainable = trainable_backbone

    inputs = layers.Input(shape=input_shape, name="image")
    # Keras EfficientNet includes its own preprocessing layer and expects
    # raw image-like values in the 0-255 range. The project tf.data pipeline
    # normalizes images to 0-1, so we scale them back here.
    x = base_model(inputs * 255.0, training=False)
    x = layers.GlobalAveragePooling2D(name="global_average_pooling")(x)
    x = layers.BatchNormalization(name="batch_norm")(x)
    x = layers.Dropout(0.35, name="dropout_035")(x)
    x = layers.Dense(
        192,
        activation="relu",
        kernel_regularizer=regularizers.l2(0.001),
        name="dense_192_relu_l2",
    )(x)
    x = layers.Dropout(0.25, name="dropout_025")(x)
    outputs = layers.Dense(num_classes, activation="softmax", dtype="float32", name="asl_softmax")(x)
    model = tf.keras.Model(inputs, outputs, name="sign4all_efficientnetb0")
    model.backbone = base_model
    return model
````

## src\models\hybrid_model.py
``python
from __future__ import annotations

import tensorflow as tf
from tensorflow.keras import layers, regularizers


def build_hybrid_model(
    num_classes: int = 27,
    image_shape: tuple[int, int, int] = (224, 224, 3),
    landmark_dim: int = 63,
    trainable_backbone: bool = False,
) -> tf.keras.Model:
    """Build hybrid CNN visual-feature + MediaPipe landmark fusion model."""
    image_input = layers.Input(shape=image_shape, name="image")
    landmark_input = layers.Input(shape=(landmark_dim,), name="landmarks")

    cnn_backbone = tf.keras.applications.MobileNetV2(
        include_top=False,
        weights="imagenet",
        input_shape=image_shape,
    )
    cnn_backbone.trainable = trainable_backbone

    image_features = tf.keras.applications.mobilenet_v2.preprocess_input(image_input * 255.0)
    image_features = cnn_backbone(image_features, training=False)
    image_features = layers.GlobalAveragePooling2D(name="visual_features")(image_features)
    image_features = layers.Dense(128, activation="relu", name="visual_dense_128")(image_features)

    landmark_features = layers.Dense(128, activation="relu", name="landmark_dense_128")(landmark_input)
    landmark_features = layers.BatchNormalization(name="landmark_bn")(landmark_features)
    landmark_features = layers.Dropout(0.2, name="landmark_dropout")(landmark_features)

    fused = layers.Concatenate(name="feature_fusion")([image_features, landmark_features])
    fused = layers.Dense(
        256,
        activation="relu",
        kernel_regularizer=regularizers.l2(0.001),
        name="fusion_dense_256",
    )(fused)
    fused = layers.BatchNormalization(name="fusion_bn")(fused)
    fused = layers.Dropout(0.35, name="fusion_dropout_035")(fused)
    fused = layers.Dense(128, activation="relu", name="fusion_dense_128")(fused)
    fused = layers.Dropout(0.2, name="fusion_dropout_020")(fused)

    outputs = layers.Dense(num_classes, activation="softmax", dtype="float32", name="asl_softmax")(fused)
    model = tf.keras.Model([image_input, landmark_input], outputs, name="sign4all_hybrid_fusion")
    model.backbone = cnn_backbone
    return model
````

## src\models\mediapipe_mlp_model.py
``python
from __future__ import annotations

import tensorflow as tf
from tensorflow.keras import layers


def build_mediapipe_mlp_model(num_classes: int = 27, input_dim: int = 63) -> tf.keras.Model:
    """Build the primary MediaPipe 21-landmark MLP classifier."""
    inputs = layers.Input(shape=(input_dim,), name="landmarks_63")

    x = layers.Dense(256, name="dense_256")(inputs)
    x = layers.BatchNormalization(name="bn_256")(x)
    x = layers.ReLU(name="relu_256")(x)
    x = layers.Dropout(0.3, name="dropout_030")(x)

    x = layers.Dense(128, name="dense_128")(x)
    x = layers.BatchNormalization(name="bn_128")(x)
    x = layers.ReLU(name="relu_128")(x)
    x = layers.Dropout(0.2, name="dropout_020_a")(x)

    x = layers.Dense(64, name="dense_64")(x)
    x = layers.BatchNormalization(name="bn_64")(x)
    x = layers.ReLU(name="relu_64")(x)
    x = layers.Dropout(0.2, name="dropout_020_b")(x)

    outputs = layers.Dense(num_classes, activation="softmax", dtype="float32", name="asl_softmax")(x)
    return tf.keras.Model(inputs, outputs, name="sign4all_mediapipe_mlp")
````

## src\models\mobilenetv2_model.py
``python
from __future__ import annotations

import tensorflow as tf
from tensorflow.keras import layers, regularizers


def build_mobilenetv2_model(
    num_classes: int = 27,
    input_shape: tuple[int, int, int] = (224, 224, 3),
    trainable_backbone: bool = False,
) -> tf.keras.Model:
    """Build MobileNetV2 transfer-learning ASL classifier."""
    base_model = tf.keras.applications.MobileNetV2(
        include_top=False,
        weights="imagenet",
        input_shape=input_shape,
    )
    base_model.trainable = trainable_backbone

    inputs = layers.Input(shape=input_shape, name="image")
    x = tf.keras.applications.mobilenet_v2.preprocess_input(inputs * 255.0)
    x = base_model(x, training=False)
    x = layers.GlobalAveragePooling2D(name="global_average_pooling")(x)
    x = layers.BatchNormalization(name="batch_norm")(x)
    x = layers.Dropout(0.3, name="dropout_030")(x)
    x = layers.Dense(
        128,
        activation="relu",
        kernel_regularizer=regularizers.l2(0.001),
        name="dense_128_relu_l2",
    )(x)
    x = layers.Dropout(0.2, name="dropout_020")(x)
    outputs = layers.Dense(num_classes, activation="softmax", dtype="float32", name="asl_softmax")(x)

    model = tf.keras.Model(inputs, outputs, name="sign4all_mobilenetv2")
    model.backbone = base_model
    return model


def compile_mobilenetv2_model(model: tf.keras.Model, learning_rate: float = 0.001) -> tf.keras.Model:
    model.compile(
        optimizer=tf.keras.optimizers.Adam(learning_rate=learning_rate, clipnorm=1.0),
        loss="sparse_categorical_crossentropy",
        metrics=["accuracy", tf.keras.metrics.SparseTopKCategoricalAccuracy(k=3, name="top3_accuracy")],
    )
    return model
````

## src\models\README.md
``markdown
# Deep Learning Model Code

This folder contains the deep learning architectures separately for easy review:

- `mobilenetv2_model.py` - MobileNetV2 transfer learning
- `efficientnetb0_model.py` - EfficientNetB0 transfer learning
- `densenet121_model.py` - DenseNet121 transfer learning
- `resnet_model.py` - custom ResNet18/ResNet34
- `mediapipe_mlp_model.py` - primary 63-landmark MLP classifier
- `hybrid_model.py` - CNN + MediaPipe landmark fusion
- `builders.py` - central factory used by the training CLI

Training command examples:

```powershell
python scripts\train_model.py --model mobilenetv2
python scripts\train_model.py --model efficientnetb0
python scripts\train_model.py --model densenet121
python scripts\train_model.py --model resnet18
python scripts\train_model.py --model resnet34
python scripts\train_model.py --model mediapipe_mlp
python scripts\train_model.py --model hybrid
```
````

## src\models\resnet_model.py
``python
from __future__ import annotations

import tensorflow as tf
from tensorflow.keras import layers


def residual_block(x: tf.Tensor, filters: int, stride: int = 1, block_name: str = "block") -> tf.Tensor:
    """Residual block with an identity/projection skip connection."""
    shortcut = x

    y = layers.Conv2D(filters, 3, strides=stride, padding="same", use_bias=False, name=f"{block_name}_conv1")(x)
    y = layers.BatchNormalization(name=f"{block_name}_bn1")(y)
    y = layers.ReLU(name=f"{block_name}_relu1")(y)
    y = layers.Conv2D(filters, 3, padding="same", use_bias=False, name=f"{block_name}_conv2")(y)
    y = layers.BatchNormalization(name=f"{block_name}_bn2")(y)

    if shortcut.shape[-1] != filters or stride != 1:
        shortcut = layers.Conv2D(filters, 1, strides=stride, padding="same", use_bias=False, name=f"{block_name}_projection")(shortcut)
        shortcut = layers.BatchNormalization(name=f"{block_name}_projection_bn")(shortcut)

    y = layers.Add(name=f"{block_name}_skip_connection")([shortcut, y])
    return layers.ReLU(name=f"{block_name}_relu_out")(y)


def build_resnet_model(
    version: str = "resnet18",
    num_classes: int = 27,
    input_shape: tuple[int, int, int] = (224, 224, 3),
) -> tf.keras.Model:
    """Build custom ResNet18 or ResNet34 for ASL recognition."""
    block_layouts = {
        "resnet18": [2, 2, 2, 2],
        "resnet34": [3, 4, 6, 3],
    }
    if version not in block_layouts:
        raise ValueError("version must be 'resnet18' or 'resnet34'")

    inputs = layers.Input(shape=input_shape, name="image")
    x = layers.Conv2D(64, 7, strides=2, padding="same", use_bias=False, name="stem_conv")(inputs)
    x = layers.BatchNormalization(name="stem_bn")(x)
    x = layers.ReLU(name="stem_relu")(x)
    x = layers.MaxPool2D(3, strides=2, padding="same", name="stem_pool")(x)

    for stage, blocks in enumerate(block_layouts[version]):
        filters = 64 * (2**stage)
        for block in range(blocks):
            stride = 2 if stage > 0 and block == 0 else 1
            x = residual_block(x, filters, stride=stride, block_name=f"stage{stage + 1}_block{block + 1}")

    x = layers.GlobalAveragePooling2D(name="global_average_pooling")(x)
    x = layers.BatchNormalization(name="head_bn")(x)
    x = layers.Dropout(0.35, name="head_dropout")(x)
    outputs = layers.Dense(num_classes, activation="softmax", dtype="float32", name="asl_softmax")(x)
    return tf.keras.Model(inputs, outputs, name=f"sign4all_{version}")
````

## src\preprocessing\__init__.py
``python
"""Dataset and landmark preprocessing."""
````

## src\preprocessing\dataset.py
``python
from __future__ import annotations

import json
from pathlib import Path

import numpy as np
import pandas as pd
import tensorflow as tf
from PIL import Image
from sklearn.model_selection import train_test_split
from sklearn.utils.class_weight import compute_class_weight

from src.utils.constants import CLASSES, IMAGE_EXTENSIONS


def canonical_label(name: str) -> str:
    return "SPACE" if name.strip().lower() in {"space", "blank"} else name.strip().upper()


def scan_dataset(dataset_dir: str | Path) -> pd.DataFrame:
    dataset_dir = Path(dataset_dir)
    rows = []
    for class_dir in dataset_dir.iterdir():
        if not class_dir.is_dir():
            continue
        label = canonical_label(class_dir.name)
        if label not in CLASSES:
            continue
        for file_path in class_dir.rglob("*"):
            if file_path.suffix.lower() in IMAGE_EXTENSIONS:
                rows.append({"path": str(file_path), "label": label})
    if not rows:
        raise FileNotFoundError(f"No images found under {dataset_dir}")
    return pd.DataFrame(rows).sort_values(["label", "path"]).reset_index(drop=True)


def remove_corrupted_images(df: pd.DataFrame) -> tuple[pd.DataFrame, list[str]]:
    valid_rows = []
    corrupted = []
    for row in df.to_dict("records"):
        try:
            with Image.open(row["path"]) as image:
                image.verify()
            valid_rows.append(row)
        except Exception:
            corrupted.append(row["path"])
    return pd.DataFrame(valid_rows), corrupted


def split_dataframe(
    df: pd.DataFrame,
    validation_split: float,
    test_split: float,
    seed: int,
) -> tuple[pd.DataFrame, pd.DataFrame, pd.DataFrame]:
    train_df, temp_df = train_test_split(
        df, test_size=validation_split + test_split, stratify=df["label"], random_state=seed
    )
    relative_test = test_split / (validation_split + test_split)
    val_df, test_df = train_test_split(
        temp_df, test_size=relative_test, stratify=temp_df["label"], random_state=seed
    )
    return train_df.reset_index(drop=True), val_df.reset_index(drop=True), test_df.reset_index(drop=True)


def class_indices(classes: list[str] | None = None) -> dict[str, int]:
    classes = classes or CLASSES
    return {label: idx for idx, label in enumerate(classes)}


def save_class_indices(path: str | Path, indices: dict[str, int]) -> None:
    path = Path(path)
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text(json.dumps(indices, indent=2), encoding="utf-8")


def dataframe_to_dataset(
    df: pd.DataFrame,
    indices: dict[str, int],
    image_size: tuple[int, int],
    batch_size: int,
    training: bool,
) -> tf.data.Dataset:
    paths = df["path"].astype(str).values
    labels = df["label"].map(indices).astype(np.int32).values
    ds = tf.data.Dataset.from_tensor_slices((paths, labels))

    def load_image(path: tf.Tensor, label: tf.Tensor) -> tuple[tf.Tensor, tf.Tensor]:
        image = tf.io.read_file(path)
        image = tf.image.decode_image(image, channels=3, expand_animations=False)
        image = tf.image.resize(image, image_size)
        image = tf.cast(image, tf.float32) / 255.0
        return image, label

    ds = ds.map(load_image, num_parallel_calls=tf.data.AUTOTUNE)
    if training:
        ds = ds.shuffle(min(len(df), 4096), reshuffle_each_iteration=True)
        ds = ds.map(lambda x, y: (augment_image(x), y), num_parallel_calls=tf.data.AUTOTUNE)
    return ds.batch(batch_size).prefetch(tf.data.AUTOTUNE)


def augment_image(image: tf.Tensor) -> tf.Tensor:
    image = tf.image.random_brightness(image, 0.2)
    image = tf.image.random_contrast(image, 0.85, 1.15)
    image = tf.image.resize_with_crop_or_pad(image, 236, 236)
    image = tf.image.random_crop(image, size=(224, 224, 3))
    return tf.clip_by_value(image, 0.0, 1.0)


def compute_weights(df: pd.DataFrame, indices: dict[str, int]) -> dict[int, float]:
    y = df["label"].map(indices).astype(int).values
    weights = compute_class_weight(class_weight="balanced", classes=np.unique(y), y=y)
    return {int(cls): float(weight) for cls, weight in zip(np.unique(y), weights)}
````

## src\preprocessing\landmarks.py
``python
from __future__ import annotations

from dataclasses import dataclass
from pathlib import Path

import cv2
import mediapipe as mp
import numpy as np
import pandas as pd
from tqdm import tqdm


try:
    mp_hands_module = mp.solutions.hands
    mp_drawing_module = mp.solutions.drawing_utils
except AttributeError:
    from mediapipe.python.solutions import drawing_utils as mp_drawing_module
    from mediapipe.python.solutions import hands as mp_hands_module


@dataclass
class LandmarkResult:
    vector: np.ndarray | None
    bbox: tuple[int, int, int, int] | None


class HandLandmarkExtractor:
    def __init__(self, static_image_mode: bool = False, max_num_hands: int = 1, min_detection_confidence: float = 0.5):
        self.mp_hands = mp_hands_module
        self.mp_draw = mp_drawing_module
        self.hands = self.mp_hands.Hands(
            static_image_mode=static_image_mode,
            max_num_hands=max_num_hands,
            min_detection_confidence=min_detection_confidence,
            min_tracking_confidence=0.5,
        )

    def close(self) -> None:
        self.hands.close()

    def extract(self, image_bgr: np.ndarray) -> LandmarkResult:
        height, width = image_bgr.shape[:2]
        image_rgb = cv2.cvtColor(image_bgr, cv2.COLOR_BGR2RGB)
        results = self.hands.process(image_rgb)
        if not results.multi_hand_landmarks:
            return LandmarkResult(None, None)

        landmarks = results.multi_hand_landmarks[0]
        points = np.array([[lm.x, lm.y, lm.z] for lm in landmarks.landmark], dtype=np.float32)
        vector = normalize_landmarks(points).reshape(-1)

        xs = [lm.x * width for lm in landmarks.landmark]
        ys = [lm.y * height for lm in landmarks.landmark]
        x1, y1 = max(int(min(xs)) - 12, 0), max(int(min(ys)) - 12, 0)
        x2, y2 = min(int(max(xs)) + 12, width - 1), min(int(max(ys)) + 12, height - 1)
        return LandmarkResult(vector, (x1, y1, x2, y2))

    def draw(self, image_bgr: np.ndarray) -> np.ndarray:
        image_rgb = cv2.cvtColor(image_bgr, cv2.COLOR_BGR2RGB)
        results = self.hands.process(image_rgb)
        if results.multi_hand_landmarks:
            for hand_landmarks in results.multi_hand_landmarks:
                self.mp_draw.draw_landmarks(image_bgr, hand_landmarks, self.mp_hands.HAND_CONNECTIONS)
        return image_bgr


def normalize_landmarks(points: np.ndarray) -> np.ndarray:
    points = points.astype(np.float32).copy()
    wrist = points[0].copy()
    points -= wrist
    scale = np.linalg.norm(points[9]) + 1e-6
    points /= scale
    return points


def build_landmark_table(df: pd.DataFrame, output_csv: str | Path) -> pd.DataFrame:
    extractor = HandLandmarkExtractor(static_image_mode=True)
    rows = []
    try:
        for row in tqdm(df.to_dict("records"), desc="Extracting landmarks"):
            image = cv2.imread(row["path"])
            if image is None:
                continue
            result = extractor.extract(image)
            if result.vector is None:
                continue
            item = {"path": row["path"], "label": row["label"]}
            item.update({f"f{i}": float(v) for i, v in enumerate(result.vector)})
            rows.append(item)
    finally:
        extractor.close()
    table = pd.DataFrame(rows)
    output_csv = Path(output_csv)
    output_csv.parent.mkdir(parents=True, exist_ok=True)
    table.to_csv(output_csv, index=False)
    return table
````

## src\training\__init__.py
``python
"""Training entry points."""
````

## src\training\callbacks.py
``python
from __future__ import annotations

from pathlib import Path

import tensorflow as tf


def training_callbacks(model_name: str, outputs_dir: str | Path, patience: int = 6) -> list[tf.keras.callbacks.Callback]:
    outputs_dir = Path(outputs_dir)
    checkpoint_dir = outputs_dir / "checkpoints"
    log_dir = outputs_dir / "logs" / model_name
    checkpoint_dir.mkdir(parents=True, exist_ok=True)
    log_dir.mkdir(parents=True, exist_ok=True)
    return [
        tf.keras.callbacks.EarlyStopping(monitor="val_loss", patience=patience, restore_best_weights=True),
        tf.keras.callbacks.ReduceLROnPlateau(monitor="val_loss", factor=0.2, patience=3, verbose=1),
        tf.keras.callbacks.ModelCheckpoint(
            filepath=str(checkpoint_dir / f"{model_name}_best.keras"),
            monitor="val_accuracy",
            save_best_only=True,
            verbose=1,
        ),
        tf.keras.callbacks.TensorBoard(log_dir=str(log_dir)),
        tf.keras.callbacks.CSVLogger(str(outputs_dir / "logs" / f"{model_name}_history.csv")),
    ]
````

## src\training\train.py
``python
from __future__ import annotations

import argparse
from pathlib import Path

import cv2
import numpy as np
import pandas as pd
import tensorflow as tf

from src.models.builders import build_model, compile_model
from src.preprocessing.dataset import (
    class_indices,
    compute_weights,
    dataframe_to_dataset,
    remove_corrupted_images,
    save_class_indices,
    scan_dataset,
    split_dataframe,
)
from src.preprocessing.landmarks import build_landmark_table
from src.training.callbacks import training_callbacks
from src.utils.config import ROOT, load_config
from src.utils.gpu import configure_gpu
from src.utils.logging_utils import get_logger
from src.visualization.plots import plot_history


class HybridSequence(tf.keras.utils.Sequence):
    def __init__(self, table: pd.DataFrame, indices: dict[str, int], batch_size: int, image_size: tuple[int, int], shuffle: bool = True):
        self.table = table.reset_index(drop=True)
        self.indices = indices
        self.batch_size = batch_size
        self.image_size = image_size
        self.shuffle = shuffle
        self.order = np.arange(len(self.table))
        self.on_epoch_end()

    def __len__(self) -> int:
        return int(np.ceil(len(self.table) / self.batch_size))

    def __getitem__(self, idx: int):
        rows = self.table.iloc[self.order[idx * self.batch_size : (idx + 1) * self.batch_size]]
        images, landmarks, labels = [], [], []
        for row in rows.to_dict("records"):
            image = cv2.imread(row["path"])
            image = cv2.cvtColor(cv2.resize(image, self.image_size), cv2.COLOR_BGR2RGB).astype(np.float32) / 255.0
            images.append(image)
            landmarks.append([row[f"f{i}"] for i in range(63)])
            labels.append(self.indices[row["label"]])
        return {"image": np.array(images, dtype=np.float32), "landmarks": np.array(landmarks, dtype=np.float32)}, np.array(labels, dtype=np.int32)

    def on_epoch_end(self) -> None:
        if self.shuffle:
            np.random.shuffle(self.order)


def train_image_model(model_name: str, config: dict) -> Path:
    logger = get_logger("sign4all.train", ROOT / "outputs/logs/train.log")
    configure_gpu(config["training"]["mixed_precision"])
    dataset_dir = ROOT / config["paths"]["dataset_dir"]
    outputs_dir = ROOT / config["paths"]["outputs_dir"]
    saved_dir = ROOT / config["paths"]["saved_models_dir"]
    outputs_dir.mkdir(parents=True, exist_ok=True)
    saved_dir.mkdir(parents=True, exist_ok=True)

    df, corrupted = remove_corrupted_images(scan_dataset(dataset_dir))
    if corrupted:
        pd.Series(corrupted).to_csv(outputs_dir / "corrupted_images.csv", index=False)
    indices = class_indices(config["data"]["classes"])
    save_class_indices(ROOT / config["paths"]["class_indices"], indices)
    train_df, val_df, test_df = split_dataframe(df, config["data"]["validation_split"], config["data"]["test_split"], config["project"]["seed"])
    for name, split in {"train": train_df, "val": val_df, "test": test_df}.items():
        split.to_csv(outputs_dir / f"{name}_split.csv", index=False)

    image_size = tuple(config["data"]["image_size"])
    batch_size = config["data"]["batch_size"]
    train_ds = dataframe_to_dataset(train_df, indices, image_size, batch_size, training=True)
    val_ds = dataframe_to_dataset(val_df, indices, image_size, batch_size, training=False)
    weights = compute_weights(train_df, indices)

    model = compile_model(build_model(model_name, len(indices)), config["training"]["learning_rate"], config["training"]["gradient_clipnorm"])
    logger.info("Training %s with %d train and %d validation images", model_name, len(train_df), len(val_df))
    history = model.fit(
        train_ds,
        validation_data=val_ds,
        epochs=config["training"]["epochs"],
        class_weight=weights,
        callbacks=training_callbacks(model_name, outputs_dir, config["training"]["patience"]),
    )
    plot_history(history.history, outputs_dir / "plots" / f"{model_name}_history.png")

    if hasattr(model, "backbone") and model_name in {"mobilenetv2", "efficientnetb0", "densenet121", "hybrid"}:
        model.backbone.trainable = True
        for layer in model.backbone.layers[:-30]:
            layer.trainable = False
        compile_model(model, config["training"]["fine_tune_learning_rate"], config["training"]["gradient_clipnorm"])
        fine_history = model.fit(
            train_ds,
            validation_data=val_ds,
            epochs=config["training"]["fine_tune_epochs"],
            class_weight=weights,
            callbacks=training_callbacks(f"{model_name}_finetune", outputs_dir, 4),
        )
        plot_history(fine_history.history, outputs_dir / "plots" / f"{model_name}_finetune_history.png")

    model_path = saved_dir / f"{model_name}.keras"
    model.save(model_path)
    logger.info("Saved model to %s", model_path)
    return model_path


def train_landmark_model(config: dict) -> Path:
    logger = get_logger("sign4all.train.landmarks", ROOT / "outputs/logs/train_landmarks.log")
    outputs_dir = ROOT / config["paths"]["outputs_dir"]
    saved_dir = ROOT / config["paths"]["saved_models_dir"]
    outputs_dir.mkdir(parents=True, exist_ok=True)
    saved_dir.mkdir(parents=True, exist_ok=True)

    indices = class_indices(config["data"]["classes"])

    split_files = {
        "train": outputs_dir / "train_split.csv",
        "val": outputs_dir / "val_split.csv",
        "test": outputs_dir / "test_split.csv",
    }
    if all(path.exists() for path in split_files.values()):
        train_df = pd.read_csv(split_files["train"])
        val_df = pd.read_csv(split_files["val"])
    else:
        df, corrupted = remove_corrupted_images(scan_dataset(ROOT / config["paths"]["dataset_dir"]))
        if corrupted:
            pd.Series(corrupted).to_csv(outputs_dir / "corrupted_images.csv", index=False)
        train_df, val_df, test_df = split_dataframe(
            df,
            config["data"]["validation_split"],
            config["data"]["test_split"],
            config["project"]["seed"],
        )
        train_df.to_csv(split_files["train"], index=False)
        val_df.to_csv(split_files["val"], index=False)
        test_df.to_csv(split_files["test"], index=False)

    overlap = set(train_df["path"]).intersection(set(val_df["path"]))
    if overlap:
        raise ValueError(f"Data leakage detected: {len(overlap)} files appear in both train and validation splits.")
    logger.info("Training MediaPipe MLP with %d train images and %d validation images", len(train_df), len(val_df))

    train_table = build_landmark_table(train_df, outputs_dir / "landmarks_train.csv")
    val_table = build_landmark_table(val_df, outputs_dir / "landmarks_val.csv")

    if train_table.empty or val_table.empty:
        raise ValueError("MediaPipe did not extract enough landmarks. Check hand visibility in dataset images.")

    x_train = train_table[[f"f{i}" for i in range(63)]].values.astype(np.float32)
    y_train = train_table["label"].map(indices).values.astype(np.int32)
    x_val = val_table[[f"f{i}" for i in range(63)]].values.astype(np.float32)
    y_val = val_table["label"].map(indices).values.astype(np.int32)

    model = compile_model(build_model("mediapipe_mlp", len(indices)), config["training"]["learning_rate"], config["training"]["gradient_clipnorm"])
    history = model.fit(
        x_train,
        y_train,
        validation_data=(x_val, y_val),
        epochs=config["training"]["epochs"],
        batch_size=config["data"]["batch_size"],
        callbacks=training_callbacks("mediapipe_mlp", outputs_dir, config["training"]["patience"]),
    )
    plot_history(history.history, outputs_dir / "plots" / "mediapipe_mlp_history.png")
    model_path = saved_dir / "mediapipe_mlp.keras"
    model.save(model_path)
    return model_path


def train_hybrid_model(config: dict) -> Path:
    outputs_dir = ROOT / config["paths"]["outputs_dir"]
    saved_dir = ROOT / config["paths"]["saved_models_dir"]
    outputs_dir.mkdir(parents=True, exist_ok=True)
    saved_dir.mkdir(parents=True, exist_ok=True)
    indices = class_indices(config["data"]["classes"])
    df, _ = remove_corrupted_images(scan_dataset(ROOT / config["paths"]["dataset_dir"]))
    train_df, val_df, test_df = split_dataframe(df, config["data"]["validation_split"], config["data"]["test_split"], config["project"]["seed"])
    test_df.to_csv(outputs_dir / "test_split.csv", index=False)
    train_table = build_landmark_table(train_df, outputs_dir / "hybrid_landmarks_train.csv")
    val_table = build_landmark_table(val_df, outputs_dir / "hybrid_landmarks_val.csv")
    image_size = tuple(config["data"]["image_size"])
    batch_size = config["data"]["batch_size"]
    train_seq = HybridSequence(train_table, indices, batch_size, image_size, shuffle=True)
    val_seq = HybridSequence(val_table, indices, batch_size, image_size, shuffle=False)
    model = compile_model(build_model("hybrid", len(indices)), config["training"]["learning_rate"], config["training"]["gradient_clipnorm"])
    history = model.fit(
        train_seq,
        validation_data=val_seq,
        epochs=config["training"]["epochs"],
        callbacks=training_callbacks("hybrid", outputs_dir, config["training"]["patience"]),
    )
    plot_history(history.history, outputs_dir / "plots" / "hybrid_history.png")
    model_path = saved_dir / "hybrid.keras"
    model.save(model_path)
    return model_path


def main() -> None:
    parser = argparse.ArgumentParser(description="Train SIGN4ALL models")
    parser.add_argument("--model", default="mediapipe_mlp", choices=["mobilenetv2", "efficientnetb0", "densenet121", "resnet18", "resnet34", "mediapipe_mlp", "hybrid"])
    parser.add_argument("--config", default="config/config.yaml")
    args = parser.parse_args()
    config = load_config(args.config)
    if args.model == "mediapipe_mlp":
        train_landmark_model(config)
    elif args.model == "hybrid":
        train_hybrid_model(config)
    else:
        train_image_model(args.model, config)


if __name__ == "__main__":
    main()
````

## src\utils\__init__.py
``python
"""Utility helpers."""
````

## src\utils\config.py
``python
from __future__ import annotations

from pathlib import Path
from typing import Any

import yaml


ROOT = Path(__file__).resolve().parents[2]


def load_config(config_path: str | Path = "config/config.yaml") -> dict[str, Any]:
    path = Path(config_path)
    if not path.is_absolute():
        path = ROOT / path
    with path.open("r", encoding="utf-8") as handle:
        config = yaml.safe_load(handle)
    return config


def project_path(*parts: str) -> Path:
    return ROOT.joinpath(*parts)
````

## src\utils\constants.py
``python
CLASSES = [
    "A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M",
    "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z",
    "SPACE",
]

IMAGE_EXTENSIONS = {".jpg", ".jpeg", ".png", ".bmp", ".webp"}
````

## src\utils\gpu.py
``python
from __future__ import annotations

import tensorflow as tf


def configure_gpu(mixed_precision: bool = True) -> list[str]:
    gpus = tf.config.list_physical_devices("GPU")
    for gpu in gpus:
        try:
            tf.config.experimental.set_memory_growth(gpu, True)
        except RuntimeError:
            pass
    if mixed_precision and gpus:
        tf.keras.mixed_precision.set_global_policy("mixed_float16")
    return [gpu.name for gpu in gpus]
````

## src\utils\logging_utils.py
``python
from __future__ import annotations

import logging
from pathlib import Path


def get_logger(name: str, log_file: str | Path | None = None) -> logging.Logger:
    logger = logging.getLogger(name)
    if logger.handlers:
        return logger

    logger.setLevel(logging.INFO)
    formatter = logging.Formatter("%(asctime)s | %(levelname)s | %(name)s | %(message)s")
    stream_handler = logging.StreamHandler()
    stream_handler.setFormatter(formatter)
    logger.addHandler(stream_handler)

    if log_file:
        path = Path(log_file)
        path.parent.mkdir(parents=True, exist_ok=True)
        file_handler = logging.FileHandler(path, encoding="utf-8")
        file_handler.setFormatter(formatter)
        logger.addHandler(file_handler)
    return logger
````

## src\visualization\__init__.py
``python
"""Visualization utilities."""
````

## src\visualization\gradcam.py
``python
from __future__ import annotations

from pathlib import Path

import cv2
import numpy as np
import tensorflow as tf


def find_last_conv_layer(model: tf.keras.Model) -> str:
    for layer in reversed(model.layers):
        if isinstance(layer, tf.keras.Model):
            try:
                return find_last_conv_layer(layer)
            except ValueError:
                pass
        if len(getattr(layer, "output_shape", [])) == 4 or isinstance(layer, tf.keras.layers.Conv2D):
            return layer.name
    raise ValueError("No convolution layer found for Grad-CAM")


def make_gradcam_heatmap(image_batch: np.ndarray, model: tf.keras.Model, class_index: int | None = None) -> np.ndarray:
    layer_name = find_last_conv_layer(model)
    grad_model = tf.keras.Model(model.inputs, [model.get_layer(layer_name).output, model.output])
    with tf.GradientTape() as tape:
        conv_outputs, predictions = grad_model(image_batch)
        if class_index is None:
            class_index = int(tf.argmax(predictions[0]))
        loss = predictions[:, class_index]
    grads = tape.gradient(loss, conv_outputs)
    pooled_grads = tf.reduce_mean(grads, axis=(0, 1, 2))
    conv_outputs = conv_outputs[0]
    heatmap = tf.reduce_sum(conv_outputs * pooled_grads, axis=-1)
    heatmap = tf.maximum(heatmap, 0) / (tf.reduce_max(heatmap) + 1e-8)
    return heatmap.numpy()


def save_gradcam(image_rgb: np.ndarray, heatmap: np.ndarray, output_path: str | Path, alpha: float = 0.35) -> None:
    output_path = Path(output_path)
    output_path.parent.mkdir(parents=True, exist_ok=True)
    heatmap = cv2.resize(heatmap, (image_rgb.shape[1], image_rgb.shape[0]))
    heatmap = np.uint8(255 * heatmap)
    colored = cv2.applyColorMap(heatmap, cv2.COLORMAP_JET)
    overlay = cv2.addWeighted(cv2.cvtColor(image_rgb, cv2.COLOR_RGB2BGR), 1 - alpha, colored, alpha, 0)
    cv2.imwrite(str(output_path), overlay)
````

## src\visualization\model_visualizer.py
``python
from __future__ import annotations

from pathlib import Path

import cv2
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import tensorflow as tf
from PIL import Image
from tensorflow.keras.preprocessing.image import ImageDataGenerator

from src.preprocessing.landmarks import HandLandmarkExtractor
from src.visualization.gradcam import make_gradcam_heatmap, save_gradcam
from src.visualization.plots import ensure_parent, plot_feature_maps, plot_landmark_skeleton, plot_tsne


def read_rgb_image(path: str | Path, image_size: tuple[int, int] = (224, 224)) -> np.ndarray:
    image = Image.open(path).convert("RGB").resize(image_size)
    return np.asarray(image, dtype=np.float32) / 255.0


def save_augmented_grid(
    image_path: str | Path,
    output_path: str | Path,
    image_size: tuple[int, int] = (224, 224),
) -> None:
    """Save augmentation examples without horizontal flipping."""
    output_path = ensure_parent(output_path)
    image = read_rgb_image(image_path, image_size)
    generator = ImageDataGenerator(
        rotation_range=15,
        zoom_range=0.15,
        width_shift_range=0.1,
        height_shift_range=0.1,
        brightness_range=[0.8, 1.2],
    )
    batch = generator.flow(image[None, ...], batch_size=1, shuffle=False)
    fig, axes = plt.subplots(3, 3, figsize=(8, 8))
    for ax in axes.reshape(-1):
        ax.imshow(np.clip(next(batch)[0], 0, 1))
        ax.axis("off")
    fig.suptitle("SIGN4ALL Augmentation Samples")
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)


def save_prediction_grid(
    model: tf.keras.Model,
    df: pd.DataFrame,
    classes: list[str],
    output_path: str | Path,
    model_type: str,
    limit: int = 16,
) -> None:
    """Save sample predictions for CNN, landmark, or hybrid models."""
    output_path = ensure_parent(output_path)
    sample_df = df.sample(min(limit, len(df)), random_state=11).reset_index(drop=True)
    extractor = HandLandmarkExtractor(static_image_mode=True)
    cols = 4
    rows = int(np.ceil(len(sample_df) / cols))
    fig, axes = plt.subplots(rows, cols, figsize=(12, 3 * rows))
    axes = np.array(axes).reshape(-1)
    try:
        for ax, row in zip(axes, sample_df.to_dict("records")):
            image_rgb = read_rgb_image(row["path"])
            image_bgr = cv2.cvtColor((image_rgb * 255).astype(np.uint8), cv2.COLOR_RGB2BGR)
            if model_type == "mediapipe_mlp":
                result = extractor.extract(image_bgr)
                if result.vector is None:
                    pred_label, confidence = "NO_HAND", 0.0
                else:
                    probabilities = model.predict(result.vector.reshape(1, -1), verbose=0)[0]
                    pred_idx = int(np.argmax(probabilities))
                    pred_label, confidence = classes[pred_idx], float(probabilities[pred_idx])
            elif model_type == "hybrid":
                result = extractor.extract(image_bgr)
                if result.vector is None:
                    pred_label, confidence = "NO_HAND", 0.0
                else:
                    probabilities = model.predict(
                        {"image": image_rgb[None, ...], "landmarks": result.vector.reshape(1, -1)},
                        verbose=0,
                    )[0]
                    pred_idx = int(np.argmax(probabilities))
                    pred_label, confidence = classes[pred_idx], float(probabilities[pred_idx])
            else:
                probabilities = model.predict(image_rgb[None, ...], verbose=0)[0]
                pred_idx = int(np.argmax(probabilities))
                pred_label, confidence = classes[pred_idx], float(probabilities[pred_idx])

            ax.imshow(image_rgb)
            ax.set_title(f"T:{row['label']} P:{pred_label} {confidence:.2f}")
            ax.axis("off")
    finally:
        extractor.close()
    for ax in axes[len(sample_df) :]:
        ax.axis("off")
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)


def save_cnn_visualizations(
    model: tf.keras.Model,
    image_path: str | Path,
    output_dir: str | Path,
    model_name: str,
) -> None:
    """Save Grad-CAM and feature map visualizations for CNN-style models."""
    output_dir = Path(output_dir)
    output_dir.mkdir(parents=True, exist_ok=True)
    image_rgb = read_rgb_image(image_path)

    try:
        heatmap = make_gradcam_heatmap(image_rgb[None, ...], model)
        save_gradcam((image_rgb * 255).astype(np.uint8), heatmap, output_dir / f"{model_name}_gradcam.png")
    except Exception as exc:
        # Transfer-learning models may hide convolution layers inside a nested backbone.
        nested_backbone = next((layer for layer in model.layers if isinstance(layer, tf.keras.Model)), None)
        if nested_backbone is not None:
            last_conv = next((layer for layer in reversed(nested_backbone.layers) if isinstance(layer, tf.keras.layers.Conv2D)), None)
            if last_conv is not None:
                prepared = prepare_backbone_input(image_rgb, model_name)
                grad_model = tf.keras.Model(nested_backbone.input, [last_conv.output, nested_backbone.output])
                with tf.GradientTape() as tape:
                    conv_outputs, backbone_output = grad_model(prepared[None, ...])
                    loss = tf.reduce_mean(backbone_output)
                grads = tape.gradient(loss, conv_outputs)
                weights = tf.reduce_mean(grads, axis=(0, 1, 2))
                heatmap = tf.reduce_sum(conv_outputs[0] * weights, axis=-1)
                heatmap = tf.maximum(heatmap, 0) / (tf.reduce_max(heatmap) + 1e-8)
                save_gradcam((image_rgb * 255).astype(np.uint8), heatmap.numpy(), output_dir / f"{model_name}_gradcam.png")
        else:
            print(f"Grad-CAM skipped for {model_name}: {exc}")

    feature_tensor = extract_feature_maps(model, image_rgb, model_name)
    if feature_tensor is not None:
        plot_feature_maps(feature_tensor, output_dir / f"{model_name}_feature_maps.png")


def prepare_backbone_input(image_rgb: np.ndarray, model_name: str) -> np.ndarray:
    if "mobilenet" in model_name.lower() or "hybrid" in model_name.lower():
        return tf.keras.applications.mobilenet_v2.preprocess_input(image_rgb * 255.0)
    if "densenet" in model_name.lower():
        return tf.keras.applications.densenet.preprocess_input(image_rgb * 255.0)
    return image_rgb


def extract_feature_maps(model: tf.keras.Model, image_rgb: np.ndarray, model_name: str) -> np.ndarray | None:
    top_conv = next((layer for layer in reversed(model.layers) if isinstance(layer, tf.keras.layers.Conv2D)), None)
    if top_conv is not None:
        feature_model = tf.keras.Model(model.input, top_conv.output)
        return feature_model.predict(image_rgb[None, ...], verbose=0)

    nested_backbone = next((layer for layer in model.layers if isinstance(layer, tf.keras.Model)), None)
    if nested_backbone is None:
        return None
    nested_conv = next((layer for layer in reversed(nested_backbone.layers) if isinstance(layer, tf.keras.layers.Conv2D)), None)
    if nested_conv is None:
        return None
    feature_model = tf.keras.Model(nested_backbone.input, nested_conv.output)
    prepared = prepare_backbone_input(image_rgb, model_name)
    return feature_model.predict(prepared[None, ...], verbose=0)


def save_landmark_visualization(image_path: str | Path, output_path: str | Path) -> None:
    image = cv2.imread(str(image_path))
    if image is None:
        return
    extractor = HandLandmarkExtractor(static_image_mode=True)
    try:
        result = extractor.extract(image)
        if result.vector is None:
            return
        plot_landmark_skeleton(result.vector.reshape(21, 3), output_path)
    finally:
        extractor.close()


def save_embedding_visualization(
    feature_model: tf.keras.Model,
    images: np.ndarray,
    labels: np.ndarray,
    classes: list[str],
    output_path: str | Path,
) -> None:
    features = feature_model.predict(images, verbose=0)
    features = features.reshape(features.shape[0], -1)
    plot_tsne(features, labels, classes, output_path)
````

## src\visualization\plots.py
``python
from __future__ import annotations

from pathlib import Path

import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sns
from PIL import Image
from sklearn.manifold import TSNE
from sklearn.metrics import ConfusionMatrixDisplay, auc, confusion_matrix, precision_recall_curve, roc_curve
from sklearn.preprocessing import label_binarize


def ensure_parent(path: str | Path) -> Path:
    path = Path(path)
    path.parent.mkdir(parents=True, exist_ok=True)
    return path


def plot_history(history: dict, output_path: str | Path) -> None:
    output_path = ensure_parent(output_path)
    fig, axes = plt.subplots(1, 2, figsize=(12, 4))
    axes[0].plot(history.get("accuracy", []), label="train")
    axes[0].plot(history.get("val_accuracy", []), label="validation")
    axes[0].set_title("Accuracy")
    axes[0].legend()
    axes[1].plot(history.get("loss", []), label="train")
    axes[1].plot(history.get("val_loss", []), label="validation")
    axes[1].set_title("Loss")
    axes[1].legend()
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)


def plot_class_distribution(df: pd.DataFrame, output_path: str | Path) -> None:
    output_path = ensure_parent(output_path)
    counts = df["label"].value_counts().sort_index()
    fig, ax = plt.subplots(figsize=(14, 5))
    sns.barplot(x=counts.index, y=counts.values, ax=ax)
    ax.set_title("Dataset Class Distribution")
    ax.set_xlabel("Class")
    ax.set_ylabel("Images")
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)


def plot_sample_images(df: pd.DataFrame, output_path: str | Path, samples: int = 16) -> None:
    output_path = ensure_parent(output_path)
    rows = []
    for label in sorted(df["label"].unique()):
        class_df = df[df["label"] == label]
        if not class_df.empty:
            rows.append(class_df.sample(1, random_state=7).iloc[0].to_dict())
    sample_df = pd.DataFrame(rows).head(samples)
    cols = 4
    rows = int(np.ceil(len(sample_df) / cols))
    fig, axes = plt.subplots(rows, cols, figsize=(12, 3 * rows))
    axes = np.array(axes).reshape(-1)
    for ax, (_, row) in zip(axes, sample_df.iterrows()):
        ax.imshow(Image.open(row["path"]).convert("RGB"))
        ax.set_title(row["label"])
        ax.axis("off")
    for ax in axes[len(sample_df) :]:
        ax.axis("off")
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)


def plot_landmark_skeleton(points: np.ndarray, output_path: str | Path) -> None:
    output_path = ensure_parent(output_path)
    connections = [
        (0, 1), (1, 2), (2, 3), (3, 4),
        (0, 5), (5, 6), (6, 7), (7, 8),
        (0, 9), (9, 10), (10, 11), (11, 12),
        (0, 13), (13, 14), (14, 15), (15, 16),
        (0, 17), (17, 18), (18, 19), (19, 20),
    ]
    fig, ax = plt.subplots(figsize=(5, 5))
    ax.scatter(points[:, 0], -points[:, 1], c=np.arange(21), cmap="viridis", s=55)
    for a, b in connections:
        ax.plot([points[a, 0], points[b, 0]], [-points[a, 1], -points[b, 1]], color="#22c55e", linewidth=2)
    ax.set_title("MediaPipe Landmark Skeleton")
    ax.axis("equal")
    ax.axis("off")
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)


def plot_misclassified(image_paths: list[str], y_true: list[str], y_pred: list[str], output_path: str | Path, limit: int = 16) -> None:
    output_path = ensure_parent(output_path)
    indices = [i for i, (a, b) in enumerate(zip(y_true, y_pred)) if a != b][:limit]
    if not indices:
        return
    cols = 4
    rows = int(np.ceil(len(indices) / cols))
    fig, axes = plt.subplots(rows, cols, figsize=(12, 3 * rows))
    axes = np.array(axes).reshape(-1)
    for ax, idx in zip(axes, indices):
        ax.imshow(Image.open(image_paths[idx]).convert("RGB"))
        ax.set_title(f"T:{y_true[idx]} P:{y_pred[idx]}")
        ax.axis("off")
    for ax in axes[len(indices) :]:
        ax.axis("off")
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)


def plot_feature_maps(feature_tensor: np.ndarray, output_path: str | Path, limit: int = 16) -> None:
    output_path = ensure_parent(output_path)
    feature_tensor = np.squeeze(feature_tensor)
    channels = min(feature_tensor.shape[-1], limit)
    cols = 4
    rows = int(np.ceil(channels / cols))
    fig, axes = plt.subplots(rows, cols, figsize=(12, 3 * rows))
    axes = np.array(axes).reshape(-1)
    for idx in range(channels):
        axes[idx].imshow(feature_tensor[:, :, idx], cmap="magma")
        axes[idx].set_title(f"Map {idx}")
        axes[idx].axis("off")
    for ax in axes[channels:]:
        ax.axis("off")
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)


def plot_confusion(y_true: np.ndarray, y_pred: np.ndarray, classes: list[str], output_path: str | Path) -> None:
    output_path = ensure_parent(output_path)
    cm = confusion_matrix(y_true, y_pred, labels=list(range(len(classes))))
    fig, ax = plt.subplots(figsize=(14, 12))
    ConfusionMatrixDisplay(cm, display_labels=classes).plot(ax=ax, xticks_rotation=90, colorbar=False)
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)


def plot_roc_pr(y_true: np.ndarray, probabilities: np.ndarray, classes: list[str], output_dir: str | Path) -> None:
    output_dir = Path(output_dir)
    output_dir.mkdir(parents=True, exist_ok=True)
    y_bin = label_binarize(y_true, classes=list(range(len(classes))))

    fig_roc, ax_roc = plt.subplots(figsize=(9, 7))
    fig_pr, ax_pr = plt.subplots(figsize=(9, 7))
    for idx, label in enumerate(classes):
        if y_bin[:, idx].sum() == 0:
            continue
        fpr, tpr, _ = roc_curve(y_bin[:, idx], probabilities[:, idx])
        precision, recall, _ = precision_recall_curve(y_bin[:, idx], probabilities[:, idx])
        ax_roc.plot(fpr, tpr, label=f"{label} AUC={auc(fpr, tpr):.2f}", alpha=0.7)
        ax_pr.plot(recall, precision, label=label, alpha=0.7)
    ax_roc.set_title("ROC Curves")
    ax_roc.set_xlabel("False Positive Rate")
    ax_roc.set_ylabel("True Positive Rate")
    ax_roc.legend(fontsize=7, ncol=2)
    fig_roc.tight_layout()
    fig_roc.savefig(output_dir / "roc_curves.png", dpi=180)
    plt.close(fig_roc)

    ax_pr.set_title("Precision-Recall Curves")
    ax_pr.set_xlabel("Recall")
    ax_pr.set_ylabel("Precision")
    ax_pr.legend(fontsize=7, ncol=2)
    fig_pr.tight_layout()
    fig_pr.savefig(output_dir / "precision_recall_curves.png", dpi=180)
    plt.close(fig_pr)


def plot_tsne(features: np.ndarray, labels: np.ndarray, classes: list[str], output_path: str | Path) -> None:
    output_path = ensure_parent(output_path)
    if len(features) < 5:
        return
    embedding = TSNE(n_components=2, init="pca", learning_rate="auto", perplexity=min(30, len(features) - 1)).fit_transform(features)
    fig, ax = plt.subplots(figsize=(10, 8))
    scatter = ax.scatter(embedding[:, 0], embedding[:, 1], c=labels, cmap="tab20", s=14)
    handles, _ = scatter.legend_elements()
    ax.legend(handles[: len(classes)], classes[: len(handles)], fontsize=7, ncol=2)
    ax.set_title("t-SNE Feature Embedding")
    fig.tight_layout()
    fig.savefig(output_path, dpi=180)
    plt.close(fig)
````

