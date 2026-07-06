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
