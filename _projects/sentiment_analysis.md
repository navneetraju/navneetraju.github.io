---
layout: page
title: Sentiment Analysis of Movie Reviews
img: assets/img/movie_review.png
importance: 2
category: Machine Learning
---

## Project Abstract and Scope

The project aims to classify movie reviews as positive or negative us. The reviews are preprocessed and tokenized before being fed into a deep learning model for classification.

## Dataset

The Movie Review Data is a collection of movie reviews retrieved from the imdb.com website in the early 2000s by Bo Pang and Lillian Lee. The reviews were collected and made available as part of their research on natural language processing. It consists of 2000 movie reviews, each of which has been tagged as positive or negative. The dataset is available at [Cornell University](https://www.cs.cornell.edu/people/pabo/movie-review-data/).

## Data Exploration and Pre Processing

### Pre Processing
We use a standard preprocessing pipeline for text data:
<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="assets/img/projects/sentiment_analysis/preprocessing.png" title="Preprocessing Pipeline" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Preprocessing Pipeline</figcaption>
</figure>

1. Clean Text: Remove special characters, numbers, and punctuation.
2. Lowercase: Convert all text to lowercase.
3. Stemming: Reduce words to their root form.
4. Stopwords: Remove common words that do not contribute to the meaning of the text.
5. Tokenization: Split the text into words.

### Dataset statistics

We calculate a few statistics to better understand the dataset:

```
---------------- Dataset Statistics ----------------
Feature                                  Value
-----------------------------------    ---------
Total Unique Words                     25297
Average Review Length                  2180.75
Median Review Length                   2024.5
Standard Deviation of Review Length    951.441
-----------------------------------------------------
```
<figcaption class="figure-caption text-center">Statistics of the dataset</figcaption>
<br/>


<div style="text-align: center;">
    <figure class="figure text-center" style="max-width: 60%; height: auto;">
        {% include figure.liquid loading="eager" path="assets/img/projects/sentiment_analysis/histogram.png" title="Review Length Histogram" class="img-fluid rounded z-depth-1" %}
        <figcaption class="figure-caption">Review Length Histogram</figcaption>
    </figure>
</div>

#### Tokenization and Padding
To represent each text (= data point), there are many ways. In NLP/Deep Learning terminology, this task is called tokenization. It is common to represent text using popularity/ rank of words in text. The most common word in the text will be represented as 1, the second most common word will be represented as 2, etc. 

```python
words = []
for text in x_raw:
    words += text.split()
word_counts = Counter(words)
sorted_word_counts = sorted(word_counts.items(), key=lambda x: x[1], reverse=True)
word_to_index = {word: i for i, (word, count) in enumerate(sorted_word_counts, 1)}

def text_to_sequence(text):
    words = text.split()
    sequence = [word_to_index[word] for word in words]
    return torch.tensor(sequence, dtype=torch.long)
```
<figcaption class="figure-caption text-center">Rank based tokenization</figcaption>

#### Thresholding

Selecting a review length threshold where 70% or 90% of the reviews have a length below it can help manage data preprocessing and analysis in NLP tasks. We choose either 70% or 90% as the threshold. 

```python
L = int(np.percentile(review_lengths, 70))
```
<figcaption class="figure-caption text-center">Thresholding</figcaption>

Interpretation
- 70th Percentile: This means that 70% of the reviews have a length below the value of threshold_70. It helps in focusing on the majority of the reviews without including many outliers.
- 90th Percentile: This means that 90% of the reviews have a length below the value of threshold_90. This is a more inclusive threshold, retaining more reviews but still excluding the longest ones.

#### Word Embeddings
One can use tokenized text as inputs to a deep neural network. However, a (not so) recent breakthrough in NLP suggests that more sophisticated representations of text yield better results. These sophisticated representations are called word embeddings. “Word embedding is a term used for representation of words for text analysis, typically in the form of a real-valued vector that encodes the meaning of the word such that the words that are closer in the vector space are expected to be similar in meaning.”

<figure class="figure text-center">
    {% include figure.liquid loading="eager" path="https://developers.google.com/static/machine-learning/crash-course/images/linear-relationships.svg" title="Word Embeddings" class="img-fluid rounded z-depth-1" %}
    <figcaption class="figure-caption">Word Embeddings in Vector Space</figcaption>
</figure>

Most deep learning modules (including PyTorch) provide a convenient way to convert positive integer reresentations of words into a word embedding by an “Embedding layer.” The layer accepts arguments that define the mapping of words into embeddings, including the maximum number of expected words also called the vocabulary size (e.g. the largest integer value). The layer also allows you to specify the dimension for each word vector, called the “embedding dimension.” We would like to use a word embedding layer for this project. 

Assume that we are interested in the top 5,000 words. This means that in each integer sequence that represents each document, we set to zero those integers that represent words that are not among the top 5,000 words in the document
```python
vocab_size = 5000
embedding_dim = 16
self.embedding = nn.Embedding(vocab_size, embedding_dim)
```
<figcaption class="figure-caption text-center">Embedding Layer in PyTorch</figcaption>

## Network Architectures

We experiment with different network architectures to classify the movie reviews.

*Note: We use early stopping in the below implementations to prevent overfitting. Early stopping is a technique used to avoid overfitting when training machine learning models. It works by monitoring the model's performance on a validation set and stopping training when performance starts to degrade.*

### Simple Neural Network
We start with a simple neural network with an embedding layer, followed by a fully connected layer and a sigmoid activation function.

```python
vocab_size = 5000
embedding_dim = 16

class NeuralNetwork(nn.Module):
    def __init__(self, vocab_size, embedding_dim):
        super(NeuralNetwork, self).__init__()
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        self.fc1 = nn.Linear(16, 1)

    def forward(self, x):
        out = self.embedding(x)
        # global average pooling
        out = out.mean(dim=1)
        out = self.fc1(out)
        return out
```

### LSTM

We experiment with a Long Short-Term Memory (LSTM) network to classify the movie reviews. LSTM is a type of recurrent neural network that is capable of learning long-term dependencies. It is well-suited to learn from sequences of data.

```python
class LSTM(nn.Module):
    def __init__(self, vocab_size, output_size, embedding_dim, hidden_dim, n_layers, drop_prob=0.5):
        super().__init__()
        self.output_size = output_size
        self.n_layers = n_layers
        self.hidden_dim = hidden_dim
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        self.lstm = nn.LSTM(embedding_dim, hidden_dim, n_layers,
                            dropout=drop_prob, batch_first=True)
        self.dropout = nn.Dropout(drop_prob)
        self.fc = nn.Linear(hidden_dim, output_size)
        self.sig = nn.Sigmoid()


    def forward(self, x):
        batch_size = x.size(0)
        embeds = self.embedding(x)
        lstm_out, hidden = self.lstm(embeds)
        lstm_out = lstm_out.contiguous().view(-1, self.hidden_dim)
        out = self.dropout(lstm_out)
        out = self.fc(out)
        sig_out = self.sig(out)
        sig_out = sig_out.view(batch_size, -1)
        sig_out = sig_out[:, -1]# get last batch of labels
        return sig_out.unsqueeze(1)

vocab_size = 5000+1
output_size = 1
embedding_dim = 400
hidden_dim = 256
n_layers = 2
net = LSTM(vocab_size, output_size, embedding_dim, hidden_dim, n_layers)
```

### Results

We use scikit-learn's classification report to evaluate the performance of the models.

#### Simple Neural Network

```
              precision    recall  f1-score   support

           0       0.87      0.88      0.87       300
           1       0.88      0.86      0.87       300

    accuracy                           0.87       600
   macro avg       0.87      0.87      0.87       600
weighted avg       0.87      0.87      0.87       600
```
The simple neural network performs well on the dataset. We can experiment with different hyperparameters and network architectures to improve the performance.

#### LSTM

```
              precision    recall  f1-score   support

           0       0.00      0.00      0.00       300
           1       0.50      1.00      0.67       300

    accuracy                           0.50       600
   macro avg       0.25      0.50      0.33       600
weighted avg       0.25      0.50      0.33       600
```
With the current implementation, the LSTM model does not perform better than the simple neural network. We can experiment with different hyperparameters and network architectures to improve the performance.

### References
- https://en.wikipedia.org/wiki/Word_embedding 
- https://developers.google.com/machine-learning/crash-course/embeddings/translating-to-a-lower-dimensional-space
- https://stackoverflow.com/questions/71998978/early-stopping-in-pytorch