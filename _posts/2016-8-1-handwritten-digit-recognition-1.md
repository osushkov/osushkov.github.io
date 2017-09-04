---
layout: post
title: Handwritten Digit Recognition - Part 1
---

As mentioned in the [previous post](https://thevoid.ghost.io/hand-written-digit-recognition/), I've spent some time working on the problem of hand-written digit recognition. This is a pretty fun problem and also a good domain for experimenting with neural nets. Here I'll note down some of my experiences so far, lessons learned, and plans for future improvement.

## Feed Forward Net
My first attempt was to use a simple feed-forward net with a logistic activation function. The high level overview is as follows:

* The input is a 28x28 grey scale image
* Input image is converted to a 784 dimensional floating point vector
* This vector is fed into the input layer of the neural network
* The input layer feeds into a hidden layer, consisting of 784 nodes
* Each node uses the [logistic function](https://en.wikipedia.org/wiki/Logistic_function) to compute its activation
* The hidden layer feeds into the output layer, consisting of 10 nodes, also with a logistic activation function
* The [cross-entropy loss function](https://en.wikipedia.org/wiki/Cross_entropy) is used to compute the error on training samples (60,000 labeled images)
* Back propagation is to compute the gradient of the network weights, and SGD (stochastic gradient descent) is used to adjust the weights to minimise error

I think the mini-batch sizes used were about 1000 samples, and the learning policy was a mix of momentum and per-weight learning rate scaling, combined with a fixed decay rate for the root learning rate.

Using this very straightforward algorithm I was able to train a neural net to recognise handwritten digits with about a **1.6% error rate**. I found that adding additional hidden layers did not really help, and even in some cases made the training worse. This was to be expected in many ways, but I wanted to do this exercise to get a simple baseline.

## Rectified Linear Unit
One of the problems with a simple logistic feed forward net is that it is very difficult to train a 'deep' net. Deep in this context simply means more than one hidden layer. This is due to the problem of the vanishing gradient. The error signal in a feed forward net comes from the output layer and is propagated backward through the hidden layers. The more layers this gradient signal passes through, the smaller it gets. This is partially due to the fact that the derivative of the sigmoid function is between 0 and 1. As you multiply these gradients together, the value gets smaller and smaller. One way to address this issue is to use the Rectified Linear Unit activation function (often shortened to ReLU). This function has the following equation: `y = max(0, x)`, although sometimes you will see a 'leaky' version used, which is: `y = x < 0.0 ? 0.01x : x`. The nice thing about this function is that it is (a) very efficient to calculate both the value and the derivative, and b) it does not suffer from the vanishing gradient effect to the same extent as the logistic function. 

With this in mind, I tried training a network with two hidden layers using the ReLU activation function, with the output units retaining the logistic function. This gave pretty good results, with the network configuration of `784 -> 784 -> 392 -> 10` trained for about 10,000 iterations resulted in an **error rate of ~1.3%**. This compared favourably with the single hidden layer logistic function network. Adding additional hidden layers did not seem to improve the accuracy with the setup I had.

## Gradient Descent Parameters
One of the thing I learned so far is the importance of getting the gradient descent parameters correct. Learning is done by applying back-propagation to a network on a batch sample of training data, giving you a gradient for the connection weights with respect to the loss function. This gradient is then applied to the network weights, and the process is repeated thousands of times. Simple enough. However, there are numerous ways in which a gradient can be used to update the network weights, and choosing a suitable method could be the difference between an efficiently learning network, and one that gets stuck on a plateau or oscillates wildly. I found a very good high-level summary of several popular algorithms by [Sebastian Ruder](http://sebastianruder.com/optimizing-gradient-descent/). 

For the experiments described here I used a combination of momentum and adaptive scaling of the gradients for each connection, along with some of my own little hacks. I found that I spent a fair bit of time tweaking the various hyper-parameters to get the network to learn fast without oscillating and diverging. Later on I implemented the [Adam](http://sebastianruder.com/optimizing-gradient-descent/index.html#adam) algorithm which seemed to work fairly well without the need to tweak numerous parameters. Bonus points for it being very simple to implement.

The code for these experiments can be found in my [Github repository](https://github.com/osushkov/handwriting)

