---
layout: page
title: Signature Forgery Detection Using Siamese Networks
img: assets/img/signature_forgery.png
importance: 3
category: Machine Learning
---

## Project Abstract and Scope
Institutions and businesses recognize signatures as the primary way of authenticating transactions. People sign checks, authorize documents and contracts, validate credit card transactions and verify activities through signatures. As the number of signed documents and their availability has increased tremendously, so has the growth of signature fraud. 

According to recent studies, only check fraud costs banks about $900M per year with 22% of all fraudulent checks attributed to signature fraud. Clearly, with more than 27.5 billion(according to The 2010 Federal Reserve Payments Study) checks written each year in the United States, visually comparing signatures with manual effort on the hundreds of millions of checks processed daily proves impractical.


<div class="row">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <figure class="figure">
            {% include figure.liquid loading="eager" path="assets/img/projects/3_project/real.png" title="Actual Signature" class="img-fluid rounded z-depth-1" %}
            <figcaption class="figure-caption">Actual Signature</figcaption>
        </figure>
    </div>
    <div class="col-sm mt-3 mt-md-0 text-center">
        <figure class="figure">
            {% include figure.liquid loading="eager" path="assets/img/projects/3_project/forged.png" title="Forged Signature" class="img-fluid rounded z-depth-1" %}
            <figcaption class="figure-caption">Forged Signature</figcaption>
        </figure>
    </div>
</div>

## Dataset

Contains Genuine and Forged signatures of 30 people. Each person has 5 Genuine signatures which they made themselves and 5 Forged signatures someone else made.

[Kaggle Dataset](https://www.kaggle.com/divyanshrai/handwritten-signatures)

## Methodology (Solution Architecture)

We use a deep convolutional Siamese network. Siamese convolution networks are twin networks with shared weights, which can be trained to learn the feature embeddings, where similar observations are placed in proximity and dissimilar, are placed apart.

<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/3_project/siamese.png" title="Siamese Architecture" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Siamese Architecture</figcaption>
</figure>

Triplet Loss function is used, wherein we contrive the data set in a triplet formation. This is done by taking an anchor image (genuine signature of a person) and placing it in conjunction with both a positive sample (another genuine signature of the same person) and a negative sample (a forged signature by someone else of the same person). 

<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/3_project/triplet_loss.webp" title="Triplet Loss Function" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Triplet Loss Framework</figcaption>
</figure>

This kind of framework ensures that the squared distance between two genuine signatures of the same individual is small, whereas the squared distance between a genuine and forged signature of an individual is large.

Triplet Loss function is defined as:

$$ L(A, P, N) = max(0, D(A, P) - D(A, N) + margin) $$

Where:
- \( A \) is the anchor image
- \( P \) is the positive image
- \( N \) is the negative image
- \( D(A, P) \) is the Euclidean distance between the anchor image and the positive image
- \( D(A, N) \) is the Euclidean distance between the anchor image and the negative image
- \( margin \) is a hyperparameter that is used to ensure that the network does not learn to minimize the distance between the anchor and negative images by making it zero.

## Evaluation and Metrics

In a traditional classifier, our performance would be based on the best prediction score of all the classes, leading to a confusion matrix with Precision, recall or F1 metrics.
In our situation, our model produces embeddings that we can use to compute distances, so we cannot apply the same system to evaluate our model performance. 

If the two pictures are from the same class, the distance should be “low”, if the pictures are from different classes, the distance should be “high”. So we need a threshold: if the found distance is under the threshold then it’s a “same” decision, if the distance is above the threshold then it’s a “different” decision. 

We have to choose this threshold carefully. Too low means we will have high precision but also too many false negatives. Too high and we will have too many false positives. This is a ROC curve problem. So, one metric for evaluation could be Area Under the Curve (AUC).


## Thresholding and Prediction

As mentioned previously, evaluation and prediction are not as straightforward as other simpler CNN classification models. The procedure to predict whether a signature is real or forged is as follows:
1. Ensure that for the test image you want to classify, you have an anchor image(Ref. methodology).
2. Use dist=siamese_model.predict(anchor_image,test_image) , to get the embeddings of the anchor image and test image and find the euclidean distance between the embeddings.
3. Check whether the Euclidean distance obtained is greater or less than the optimal threshold:

```
if dist < threshold:
    print("The signature is Genuine")
else:
    print("The signature is Forged")
```

To find the optimal threshold i.e the optimal distance value to classify as “Real” or “Forged” we plotted a ROC graph. But before plotting the graph we needed to obtain distance measures between (Anchor images, Real Images) and (Anchor Images, Forged Images) and obtain two arrays called pos and neg. These 2 arrays were concatenated. This concatenated array was nothing but our y_pred. This was then used in conjunction with y_true in order to evaluate the model and also to find the optimal threshold.

Using the ROC graph plotted, we used a simple formula to find the most “optimal” cut off point on the ROC graph, this optimal point, if used as a threshold for classification, must yield Low False Positive Rate(FPR) and High True Positive Rate(TPR).

The formula used to find the optimal threshold is:

```
optimal_idx = np.argmax(tpr - fpr)
optimal_threshold = thresholds[optimal_idx]
```
Where:
- tpr is True Positive Rate
- fpr is False Positive Rate
- thresholds are the thresholds obtained from the model.

### Results

##### Training Data:
AUC Score: 0.999 (3650 samples)
```
Optimal Threshold Found From ROC AUC Curve:  0.002733261790126562
Accuracy:  0.9919836956521739
F1 Score:  0.9920281043102284
```

Confusion Matrix:

<table style="border-collapse: collapse; width: 100%;">
    <tr>
        <th style="border: 1px solid black; padding: 8px; text-align: left;">&nbsp;</th>
        <th style="border: 1px solid black; padding: 8px; text-align: left;">Predicted Genuine</th>
        <th style="border: 1px solid black; padding: 8px; text-align: left;">Predicted Forged</th>
    </tr>
    <tr>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">Actual Genuine</td>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">3630</td>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">50</td>
    </tr>
    <tr>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">Actual Forged</td>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">9</td>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">3671</td>
    </tr>
</table>
<br/>

<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/3_project/train_curve.png" title="ROC AUC Curve For Training Data" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">ROC AUC Curve For Training Data</figcaption>
</figure>

##### Test Data:
AUC Score: 0.985 (920 samples)
```
Optimal Threshold Found From ROC AUC Curve:  0.0013851551339030266
Accuracy:  0.9315217391304348
F1 Score:  0.9341692789968652
```

Confusion Matrix:

<table style="border-collapse: collapse; width: 100%;">
    <tr>
        <th style="border: 1px solid black; padding: 8px; text-align: left;">&nbsp;</th>
        <th style="border: 1px solid black; padding: 8px; text-align: left;">Predicted Genuine</th>
        <th style="border: 1px solid black; padding: 8px; text-align: left;">Predicted Forged</th>
    </tr>
    <tr>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">Actual Genuine</td>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">410</td>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">50</td>
    </tr>
    <tr>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">Actual Forged</td>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">13</td>
        <td style="border: 1px solid black; padding: 8px; text-align: left;">447</td>
    </tr>
</table>
<br/>

<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/3_project/test_curve.png" title="ROC AUC Curve For Test Data" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">ROC AUC Curve For Test Data</figcaption>
</figure>