# Frameworks
## TensorFlow
* open-sourced two years ago, TensorFlow is Google's own internal framework for deep learning
* "we could have selected other prominent deep learning frameworks that are equally as effective as TensorFlow-like Theano, Torch, or Caffe — but TensorFlow has risen in popularity so quickly that it's now arguably the *de facto* industry standard for deep learning"
* but, its intrinsic *difficulty* of use, due in part to the fact that the framework is so customizable and "down to the metal" of every little detail
## Caffe
* with better expression, speed, and modularity as the focus points
* developed for *computer vision/image classification* by leveraging Convolutional Neural Networks(CNNs). Caffe is popular for its Model Zoo, which is a set of pre-trained models that doesn’t require any coding to implement.
* It is better suited for building applications as opposed to Tensorflow which fares better at research and development.
* If you are dealing with applications with text, sound or time series data, note that Caffe is *not* intended for anything other than computer-vision

## Torch
* arguably be the simplest machine learning framework to set up and get going fast and easily
* write in *lua* language

## PyTorch:  a python version
* PyTorch is essentially a *GPU* enabled drop-in replacement for *NumPy* equipped with higher-level functionality for building and training deep neural networks.
### compare
* PyTorch is better for rapid prototyping in research, for hobbyists and for small scale projects.
* TensorFlow is better for large-scale deployments, especially when cross-platform and embedded deployment is a consideration.
* However, there is still some functionality which TensorFlow supports that PyTorch doesn’t. A few features that PyTorch doesn’t have (at the time of writing) are:
  * Flipping a tensor along a dimension (np.flip, np.flipud, np.fliplr)
  * Checking a tensor for NaN and infinity (np.is_nan, np.is_inf)
  * Fast Fourier transforms (np.fft)

## Deeplearning4j
* java!

## CNTK
* Microsoft thing


## others
* Theano
* Brainstorm
* SciKit-learning: python based
