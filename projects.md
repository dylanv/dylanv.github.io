---
layout: page
title: Projects
permalink: /projects/
---

### Personal

My [Github](https://github.com/DylanV/)

#### Simple neural nets

```python
net = Sequential(Linear(28 * 28, 100), Sigmoid(), Linear(100, 10), Sigmoid(), SoftmaxCrossEntropy())
optimiser = Adam(net.parameters, net.backward)
optimiser.train(X, y, epochs=5, batch_size=10, learning_rate=1, regularisation_weight=1e-3)
```

#### Character RNNs

This was inspired by Karpathy's semminal [blog post](http://karpathy.github.io/2015/05/21/rnn-effectiveness/) on character RNNs. My focus has been
on implementing the RNN architectures from scratch in Pytorch. Both to practice my Pytorch-fu and to get a feel for ANN sequence modelling. 

I've done up to LSTM's on the wiki-text2 dataset and things seems to work pretty well.

### Academic

#### Master's Thesis

#### Undergraduate final year project