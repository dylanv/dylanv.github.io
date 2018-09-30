---
layout: page
title: Projects
permalink: /projects/
---

### Personal

My [Github](https://github.com/DylanV/) :)

#### Simple neural nets ([github](https://github.com/dylanv/simple-neural-nets))

This was a learning project to get to really grok the inner workings of neural networks. It's a minimal sequential network library with a lot of 
the basics implemented.  
It's built on numpy and allows you to easily set up and train networks. A minimal (and fairly weird) example that shows a lot of functionality would look something like this: 

```python
net = Sequential(Linear(784, 100), BatchNorm(), Sigmoid(), Dropout(),
			     Linear(100, 10), SoftmaxCrossEntropy())
optimiser = Adam(net.parameters, net.backward)
optimiser.train(X, y, epochs=5, batch_size=10, learning_rate=1, regularisation_weight=1e-3)
```

#### Character RNNs ([github](https://github.com/dylanv/RNN))

This was inspired by Karpathy's seminal [blog post](http://karpathy.github.io/2015/05/21/rnn-effectiveness/) on character RNNs. My focus has been
on implementing the RNN architectures from scratch in Pytorch. Both to practice my Pytorch-fu and to get a feel for ANN sequence modelling. 

Up to LSTM's on the wiki-text2 dataset and things seem to work pretty well. Getting presentable (if incoherent english).

### Academic

#### Bayesian Recommender Systems and Approximate Inference

* Master's Thesis
*  Developed and implemented a Bayesian recommender system based on the Latent Dirichlet Allocation (LDA) topic model.
* Explored how various inference approaches (variational inference, expectation propagation and Gibbs sampling) affect recommendation accuracy.
* Implemented Bayesian model with inference in C++ and various other recommender systems in Python. Including unit tests.

#### Parallelisation of Loopy Belief Inference Systems in PGMs

* Undergraduate final project
* Developed three concurrent approaches for concurrent loopy belief propagation inference for PGMs. Achieved a worst case speed-up of 2x over existing implementation.
* Using concurrency features added in {C++11}, implemented the concurrent approaches and integrated with the university's PGM package.
