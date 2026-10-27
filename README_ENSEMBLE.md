# Ensemble Prescription Fraud Detection System

This system now uses a hybrid approach combining **CNN (Convolutional Neural Network)** and **Random Forest** for improved accuracy in detecting fake prescriptions, especially AI-generated ones.

## What's New

### Previous System
- Used only Random Forest with hand-crafted features
- Limited accuracy on AI-generated fake prescriptions
- Relied on traditional image processing features

### New Ensemble System
- **CNN Model**: Deep learning approach that learns visual patterns automatically
- **Random Forest**: Traditional ML with hand-crafted features (edge detection, texture, etc.)
- **Ensemble**: Weighted combination of both models for better accuracy
- **Higher Accuracy**: CNN is particularly good at detecting AI-generated artifacts

## Architecture

```
Input Image
    ↓
Prescription Validation (is it a prescription?)
    ↓
Ensemble Model
    ├── CNN Model (60% weight) - Deep learning features
    └── Random Forest (40% weight) - Hand-crafted features
    ↓
Weighted Average Prediction
    ↓
Final Result (FAKE/GENUINE with confidence)
```

## Training the Models

### Option 1: Train Both Models (Recommended)

Run the unified training script:

```bash
cd "C:\Users\Kushagra\OneDrive\Attachments\Desktop\Design_Experience\prescription-detector"
python train_ensemble.py
```

This will:
1. Train the Random Forest model with hand-crafted features
2. Train the CNN model with deep learning
3. Save both models to the `models/` directory
4. Generate training history plots for CNN

### Option 2: Train Individual Models

**Train only CNN:**
```bash
python train_cnn.py
```

**Train only Random Forest:**
```bash
python train_model.py
```

### Option 3: Train with Custom Paths

```bash
python train_ensemble.py --fake-folder "path/to/fake" --real-folder "path/to/real" --model-dir "path/to/models"
```

## Requirements

The CNN training requires TensorFlow. Install it if not already installed:

```bash
pip install tensorflow tensorflow-gpu
```

For CPU-only training (slower but works):
```bash
pip install tensorflow
```

## Model Files

After training, you'll find these files in the `models/` directory:

### Random Forest
- `random_forest_model_latest.pkl` - Latest RF model
- `scaler_latest.pkl` - Feature scaler for RF
- `random_forest_model_YYYYMMDD_HHMMSS.pkl` - Timestamped backup

### CNN
- `cnn_model_latest.keras` - Latest CNN model
- `cnn_model_YYYYMMDD_HHMMSS.keras` - Timestamped backup
- `cnn_training_history_YYYYMMDD_HHMMSS.png` - Training plots
- `cnn_history_YYYYMMDD_HHMMSS.json` - Training metrics

## Using the Ensemble System

The backend automatically uses the ensemble model if both models are available. No code changes needed!

### Start the Backend

```bash
cd "C:\Users\Kushagra\OneDrive\Attachments\Desktop\Design_Experience\prescription-detector\backend"
python app.py
```

### Check Model Status

Visit `http://localhost:8000/health` to see:
- Which models are loaded
- Ensemble weights
- Model status

### API Response

The ensemble API returns additional information:

```json
{
  "prediction": "FAKE" or "GENUINE",
  "confidence": 0.85,
  "genuine_probability": 0.15,
  "fake_probability": 0.85,
  "ensemble_weights": {
    "cnn_weight": 0.6,
    "rf_weight": 0.4
  },
  "individual_predictions": {
    "cnn": {
      "genuine_probability": 0.1,
      "fake_probability": 0.9
    },
    "random_forest": {
      "genuine_probability": 0.25,
      "fake_probability": 0.75
    }
  },
  "method": "ensemble_cnn_rf"
}
```

## Customizing Ensemble Weights

If you want to adjust the weight given to each model, edit `ensemble_model.py`:

```python
# In EnsemblePrescriptionDetector.__init__
self.cnn_weight = 0.6  # Adjust this (0.0 to 1.0)
self.rf_weight = 0.4   # Adjust this (0.0 to 1.0)
```

Or adjust dynamically:

```python
detector = EnsemblePrescriptionDetector()
detector.set_weights(cnn_weight=0.7, rf_weight=0.3)
```

## Expected Performance

With both models trained:
- **CNN**: Excellent at detecting AI-generated artifacts, deepfakes, and synthetic images
- **Random Forest**: Good at detecting traditional forgery signs (compression artifacts, lighting issues)
- **Ensemble**: Combines strengths of both for higher overall accuracy

## Troubleshooting

### TensorFlow Installation Issues

If you have trouble installing TensorFlow:

1. Check Python version (3.8-3.11 recommended)
2. Install CPU version: `pip install tensorflow`
3. For GPU support, install CUDA and cuDNN first

### Out of Memory During CNN Training

If you run out of memory:

1. Reduce batch size in `train_cnn.py` (line ~368)
2. Reduce image size in `PrescriptionCNNTrainer.__init__`
3. Close other applications

### Model Not Loading

If the backend doesn't load the models:

1. Check that model files exist in `models/` directory
2. Check backend logs for error messages
3. Ensure TensorFlow is installed
4. Verify file paths are correct

### Poor Performance

If accuracy is still low:

1. **Collect more fake samples** - This is the most important factor
2. Ensure your fake samples are diverse (different AI generators, styles)
3. Balance your dataset (similar number of fake and real samples)
4. Try adjusting ensemble weights
5. Train CNN for more epochs (increase `epochs=50` to `epochs=100`)

## Next Steps

1. **Train the ensemble models**: Run `python train_ensemble.py`
2. **Restart the backend**: Stop and restart the Flask/FastAPI server
3. **Test with images**: Upload both real and fake prescriptions
4. **Monitor performance**: Check the confidence scores and individual model predictions
5. **Iterate**: Add more training data if needed

## Additional Benefits

- **Robustness**: If one model fails, the other can still provide predictions
- **Explainability**: See individual model predictions to understand why a decision was made
- **Flexibility**: Can adjust weights based on performance
- **Future-proof**: Easy to add more models (e.g., Vision Transformer) to the ensemble

## Need Help?

If you encounter issues:
1. Check the error messages in the console
2. Verify all dependencies are installed
3. Ensure your dataset has both fake and real images
4. Try training with a smaller dataset first to test the pipeline
