# Experiments: Regularisation and Tracking

## 1. Objective

The objective was to study overfitting and evaluate regularisation
techniques on the Intel Image Classification dataset.

## 2. Dataset and Setup

- Dataset: Intel Image Classification
- Classes: buildings, forest, glacier, mountain, sea, street
- Total images: 14034
- Training images: 11227
- Validation images: 2807
- Image size: 128 x 128
- Batch size: 64
- Optimiser: Adam
- Learning rate: 0.001
- Random seed: 42
- Device: cuda

An 80/20 training-validation split was used.

## 3. Overfitting Experiment

The baseline CNN was trained for 20 epochs.

Training loss continued to decrease, while validation loss
reached its minimum at epoch 16 (0.4784).
After that, validation loss fluctuated and generally remained higher.

This indicates a modest overfitting pattern.

## 4. Regularisation Techniques

The following techniques were tested independently:

1. Dropout: dropout probability of 0.4.
2. Weight decay: 0.001.
3. Early stopping: patience of 3 epochs.
4. Data augmentation: random horizontal flip and rotation of 10 degrees.

## 5. Results

| Technique         | Setting                          |   Training Accuracy |   Validation Accuracy |   Best Validation Loss |   Best Epoch |
|:------------------|:---------------------------------|--------------------:|----------------------:|-----------------------:|-------------:|
| Baseline          | No regularisation                |              0.8324 |                0.8276 |                 0.4784 |           16 |
| Dropout           | Dropout = 0.4                    |              0.7908 |                0.8133 |                 0.5433 |           15 |
| Weight Decay      | Weight decay = 0.001             |              0.7952 |                0.7984 |                 0.5640 |           14 |
| Early Stopping    | Patience = 3                     |              0.8324 |                0.8276 |                 0.4784 |           16 |
| Data Augmentation | Horizontal flip + rotation (10°) |              0.8065 |                0.8133 |                 0.5271 |           15 |
| Combined          | Early stopping + Dropout (0.4)   |              0.8016 |                0.8251 |                 0.5016 |           20 |

Accuracy values are measured at the epoch with the lowest validation loss.

## 6. Combined Experiment

The two techniques selected were early stopping and dropout.

The combined model achieved validation accuracy of
82.51%.

It did not outperform the baseline model. Combining regularisation
methods did not provide an improvement in this experiment.

## 7. Reproducibility

All experiments used random seed 42.

The combined experiment was repeated with the same configuration.

- First validation loss: 0.5016003054
- Second validation loss: 0.5016003054
- First validation accuracy: 0.8250801568
- Second validation accuracy: 0.8250801568

Both runs produced identical results in this Colab environment.

## 8. What Worked

- The baseline achieved the highest validation accuracy.
- Early stopping identified the best validation checkpoint.
- Dropout reduced the training-validation performance gap.
- Repeated runs produced identical results.

## 9. What Did Not Work

- Weight decay did not improve validation accuracy.
- Dropout and data augmentation individually performed below baseline.
- Combining early stopping and dropout did not beat the baseline.

## 10. Next Experiments

- Experiment with different dropout probabilities.
- Test different weight decay values.
- Try stronger or more varied image augmentation.
- Train for more epochs and compare learning curves.
- Evaluate the best model on a separate test dataset.

## 11. Limitations

- Results depend on the selected random split and training configuration.
- Only one seed was used for the main comparison.
- Validation accuracy is not a substitute for final test-set evaluation.
- The experiments used a relatively small CNN rather than a pretrained model.
