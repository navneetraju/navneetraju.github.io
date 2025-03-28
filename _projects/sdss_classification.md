---
layout: page
title: SDSS Data Classification
img: assets/img/projects/sdss/thumbnail.png
importance: 4
category: Machine Learning
---

This project leverages the extensive Sloan Digital Sky Survey (SDSS) dataset to classify astronomical objects. It showcases an end-to-end machine learning pipeline—from data preprocessing to model evaluation—and demonstrates how modern algorithms can be effectively applied to astronomical data.

## Introduction

The Sloan Digital Sky Survey (SDSS) is one of the most comprehensive astronomical surveys ever undertaken. Capturing high-quality images and spectra of millions of celestial objects, the SDSS dataset has significantly enhanced our understanding of the universe. This project aims to accurately classify these celestial objects (stars, galaxies, quasars) using machine learning techniques.

## The Dataset: Understanding SDSS

### Overview:

- **Data Type:** Includes multi-band photometric data (u, g, r, i, z) and spectroscopic data.
- **Richness:** Millions of data points ideal for extensive astrophysical research.
- **Challenges:** High dimensionality, class imbalance, and data complexity.

## Problem Statement

This project tackles the classification of astronomical objects, specifically addressing:

- **Data Imbalance:** Addressing uneven representation across classes.
- **Dimensionality Reduction:** Managing high-dimensional data effectively.
- **Model Accuracy:** Developing robust models capable of generalizing well.

## Methodology

### Data Preprocessing

- **Cleaning and Normalization:** Handled missing values and normalized features.
- **Train-Test Split:** Ensured unbiased model evaluation.
- **Imbalance Handling:** Utilized Synthetic Minority Over-sampling Technique (SMOTE), resulting in a balanced dataset of 6890 samples (3445 per class).

### Correlation Matrix

<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/sdss/correlation.png" title="Correlation Heatmap" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Correlation Heatmap</figcaption>
</figure>

- **Dimensionality Reduction:** Applied Principal Component Analysis (PCA) for efficient feature representation.

### Model Development

- **Neural Network Architecture:**
  - Input Layer: 16 neurons with ReLU activation
  - Hidden Layers: 32 and 16 neurons, both with ReLU activation
  - Output Layer: 1 neuron with Sigmoid activation

<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/sdss/arch.png" title="FC Network Architecture" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">FC Network Architecture</figcaption>
</figure>

- **Training:** Optimized model through extensive training over 80,000 epochs with progressive improvements in accuracy and reduction in loss.

### Training Loss and Accuracy

<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/sdss/training.png" title="Training loss and Accuracy plot" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Training loss and Accuracy plot</figcaption>
</figure>

- **Evaluation:** Assessed model using accuracy, precision, recall, and F1-score.

## Model and Metrics

The neural network model effectively addresses data complexity, feature-richness, and class imbalance, enhanced by:

- **Feature Engineering:** Additional feature extraction to boost predictive power.
- **SMOTE:** Balanced dataset representation.
- **PCA:** Reduced dimensionality for computational efficiency.

### Evaluation Metrics

- **Accuracy:** 93% on test set
- **Precision and Recall:** High precision and recall for both classes
- **F1-Score:** 93% macro and weighted averages

#### Classification Report (Test Set):

```
              precision    recall  f1-score   support

           0       0.91      0.94      0.93      1034
           1       0.94      0.91      0.93      1033

    accuracy                           0.93      2067
   macro avg       0.93      0.93      0.93      2067
weighted avg       0.93      0.93      0.93      2067
```

## Results

The project successfully demonstrated:

- **High Accuracy:** Effective classification of astronomical objects.
- **Balanced Class Performance:** Maintained performance across classes, addressing imbalance effectively.
- **Scalability:** Adaptability to larger and future datasets.

## Conclusion and Future Work

The successful classification demonstrates machine learning's potential in astronomical research. Future improvements could include:

- **Advanced Models:** Incorporation of deep learning and ensemble methods.
- **Additional Features:** Integration of broader astrophysical data.
- **Real-Time Applications:** Adaptation for real-time classification in observational astronomy.

For comprehensive code and further documentation, visit the [GitHub repository](https://github.com/navneetraju/SDSS-Data-Classification/tree/master).
