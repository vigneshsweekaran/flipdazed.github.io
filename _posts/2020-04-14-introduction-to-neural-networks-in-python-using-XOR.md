---
layout: post
title: "Introduction to Neural Nets in Python with XOR"
menutitle: "Introduction to Neural Nets in Python with XOR"
date: 2020-04-13 21:35:00 +0000
tags: Python Tutorial Neural Networks Machine Learning XOR
category: Python Tutorial
author: am
published: true
redirect_from: "/2020-04-13-introduction-to-neural-networks-in-python-using-XOR/"
language: EN
comments: true
---

## Contents
{:.no_toc}

* This will become a table of contents (this text will be scraped).
{:toc}

This is a beginners introduction. Each topic, subtopic and even sentence most likely has alternatives and improvements in modern literature. If a topic interests you hit you favourite search engine and you will find a lot more information. 

# Expected background

- **Programming** The level of programming expected is beginner. You should be comfortable with lists, data structures, iteration and be familiar with numpy.
- **Maths** The level of maths is GCSE/AS level (upper-high school). You should be able to taken derivatives of exponential functions and be familiar with the chain-rule.

# Theory 

## The XOR function
The XOR function is the simplest (afaik) non-linear function.
Is is impossible to separate True results from the False results using a linear function.


```python
def xor(x1, x2):
    """returns XOR"""
    return bool(x1) != bool(x2)

x = np.array([[0,0],[0,1],[1,0],[1,1]])
y = np.array([xor(*x) for x in inputs])
```

This is clear on a plot

```python
import pandas as pd
import seaborn as sns
sns.set()

data = pd.DataFrame(x, columns=['x1', 'x2'])
data['xor'] = y
sns.scatterplot(data=data, x='x1', y='x2', style='xor', hue='xor', s=100)
```

<figure>
  <img src="{{ site.baseurl }}/media{{page.redirect_from}}xor_0.png" />
  <figcaption>No straight line (plane) can entirely separate all the True values from the False values.</figcaption>
</figure>

A neural network is essentially a series of hyperplanes (a plane in N dimensions) that group / separate regions in the target hyperplane.

Two lines is all it would take to separate the True values from the False values in the XOR gate.

# The Perceptron

There's lots of good articles about perceptrons. To quickly summarise, a perceptron is essentially a method of separating a manifold with a hyperplane. This is just drawing a straight line to separate an n-dimensional space into two regions: True or False. I will interchangeably refer to these as neurons or perceptrons. Arguablly different, they are basically the same thing.

<figure>
  <img src="{{ site.baseurl }}/media{{page.redirect_from}}perceptron_0.png" />
  <figcaption>A perceptron model from [stackexchange](https://tex.stackexchange.com/questions/104334/tikz-diagram-of-a-perceptron)</figcaption>
</figure>

### Activation functions

The way our brains work is like a sort of step function. Neurons fires a 1 if there is enough build up of voltage else it doesn't fire (i.e a zero). We aim, via the percepton, to recreate this behaviour.

The problem with a step function is that they are discontinuous. This creates problems with the practicality of the mathematics *(talk to any derivatives trader about the problems in hedging barrier options at the money)*. Thus we tend to use a smooth functions, the [sigmoid](https://en.wikipedia.org/wiki/Sigmoid_function), which is infinitely differentiable, allowing us to easily do calculus with our model.

```python
def sigmoid(x):
    return 1/(1 + np.exp(-x))
```

### Hyperplanes

A single perceptron, therefore, cannot separate our XOR gate because it can only draw one straight line.

<figure>
  <img src="{{ site.baseurl }}/media{{page.redirect_from}}xor_1.png" />
  <figcaption>How do we draw two straight lines? (from [Kevin Swingler](http://www.cs.stir.ac.uk/courses/ITNP4B/lectures/kms/2-Perceptrons.pdf) via [Lucas Araújo](https://medium.com/@lucaspereira0612/solving-xor-with-a-single-perceptron-34539f395182))</figcaption>
</figure>

The trick is to realise that we can just logically stack two perceptrons. Two perceptrons that will draw straight lines, and another perceptron that serves to combine these two separate signals into a single signal that just has to differntiate between a single True / False boundary.

<figure>
  <img src="{{ site.baseurl }}/media{{page.redirect_from}}xor_2.png" />
  <figcaption>The XOR gate can be created by the following combination of a NOT AND gate and an OR gate (from [blog.abhranil.net](https://blog.abhranil.net/2015/03/03/training-neural-networks-with-genetic-algorithms/))</figcaption>
</figure>

## Learning Algorithm

The "knowledge" of a neural network is all contained in the learned parameters whcih are the weights and bias. The weights are multiplied to each signal sent by their respective perceptrons and the bias are added. They change the signal from a 1/0 an input like $y(x) = wx + b$ where here $x$ is the sigmoid that approximately sends outs 1/0, $w$ is the weight and $b$ is the bias.

The backpropagation algorithm (backprop.) is the key method by which we seqeuntially adjust the weights by backpropagating the errors from the final output neuron.

We define define the error as anything that will decrease as we approach the target distribution. Let $E$ is the total error given by 

$$E = \frac{1}{2}(y - y_{o})$$

where $y_o$ is the result of the output layer (the prediction) and $y$ is the true value given in the training data.

Later we will require the derivative of this function so we can add in a factor of 0.5 which simplifies the derivative.

```python
def error(target, prediction):
    return .5*(target - prediction)**2
```

The learning algorithm consists of the following steps:

1. Randomly initialise bias and weights
2. Iterate the training data
  a. Forward propagate: Calculate the neural net the output
  b. Compute a "loss function"
  c. Backwards propagate: Calculate the gradients with respect to the weights and bias
  d. Adjust weights and bias by gradient descent
3. Exit when error is minimised to some criteria

Note that here we are trying to replicate the exact functional form of the input data. This is not probabilistic data so we do not need a train / validation / test split as overtraining here is actually the aim.


## Back propagation

We want to find the minimum loss given a set of parameters (the weights and biases). Recalling some AS level maths, we can find the minima of a function by minimising the gradient (each minima has zero gradient). Also luckily for us, this problem has no local minima so we don't need to do any funny business to guarantee convergence. 

Real world problems require stoastic gradient descents which "jump about" as they descend giving them the ability to find the global minima given a long enough time.

We therefore have several quantitites that require calculation
$\frac{\partial E}{\partial w_{o}}$,
$\frac{\partial E}{\partial w_{h}}$,
$\frac{\partial E}{\partial b_{o}}$
and $\frac{\partial E}{\partial w_{h}}$ where $h$ and $o$ denote hidden and output layers and $E$ is the total error given by $\frac{1}{2}(y - y_{o})$

### Output layer gradient

Starting at the output layer, and by chain rule,
$$
\frac{\partial E}{\partial w_{o}} =
\frac{\partial E}{\partial y_{o}}
\frac{\partial y_o}{\partial a_{o}}
\frac{\partial a_o}{\partial w_{o}}
$$

This is true because the output layer $y_o = \sigmoid(a_o)$ varies with respect to the activation $a_o = w_o \dot y_h + b_o$ in the output layer. The activation in the output layer varies with respect to the weights of the output layer. Recall that this is a partial derivative so we hold the bias of the output layer and the hidden layer output as constants. We can calculate all these terms.

The derivative of the Error with respect to the output layer is just 
$$
\frac{\partial E}{\partial y_{o}} = -(y - y_o)
$$
due to the sneaky extra half

```python
def error_derivative(target, prediction):
    return - target + prediction
```

The derivative of the output layer with respect to the sigmoid is 
$$
\frac{\partial y_o}{\partial a_{o}} = \frac{\partial \sigma(a_o)}{\partial a_o} = \sigma(a_o) (1 - \sigma(a_o)) = y_o (1 - y_o)
$$

```python
def sigmoid_derivative(sigmoid_result):
    return sigmoid_result * (1 - sigmoid_result)
```

The derivative of the activation function with respect to the weights is
$$
\frac{\partial a_o}{\partial w_{o}} = y_h
$$
which is just the result from the hidden layer.

The same is true for the bias (changing $w_o$ for $b_o$) except

$$\frac{\partial a_o}{\partial b_{o}} = 1$$ 

### Hidden layer gradient

For the hidden layer we just keep expanding. Instead of taking $y_h$ as a variable we now treat it as a function that varies with the output of the hidden layer. Thus instead of stopping at $\frac{\partial a_o}{\partial w_o}$ we must continue since $a_o = w_o \cdot y_h + b_o$ is a fuction of $y_h$ which is no longer a constant. 

Recall, if we vary $w_h$, $y_h = \sigmoid(a_h)$ and further, $a_h = w_h \cdot x + b$ where $x$ is the input training data.

Starting at the output layer, and by chain rule,
$$
\frac{\partial E}{\partial w_h} =
\frac{\partial E}{\partial y_o}
\frac{\partial y_o}{\partial a_o}
\frac{\partial a_o}{\partial y_h}
\frac{\partial y_h}{\partial a_h}
\frac{\partial a_h}{\partial w_h}
$$

the three extra derivatives we need to calculate are
$$
\frac{\partial a_o}{\partial y_h} = w_o
$$

then recall the derivative of the sigmoid from before
$$
\frac{\partial y_h}{\partial a_h} = y_h (1 - y_h)
$$

and finally
$$
\frac{\partial a_h}{\partial w_h} = x
$$
and again the bias are the same except $\frac{\partial a_h}{\partial b_h} = 1$


### Parameter updates

Let all the parameters be defined in the vector $\theta = (w_o, w_h, b_o, b_h)$. Then they are all updated like

$$
\theta \leftarrow \theta - \alpha \frac{\partial E}{\partial \theta} 
$$

where $\alpha$ is the learning rate that is fixed at some constant (mysteriously, usually about 0.02) and this defines how fast we descend the gradient. Too fast and we overshoot and too small and it takes too long.

Explicitly for $w_o, b_o$ this would be,

$$
w_o \to w_o + \alpha y^T_h \cdot (y_o - y) y_o (1 - y_o)\\
b_o \to b_o + \alpha (y_o - y) y_o (1 - y_o)
$$

where the $y_h^T$ denotes the transpose of the vector $y_h$. I was a bit underhand in the maths to simplify, but as we will see there is some care required with transposing of dot products when multiplying the weights and layers.

# Implementation

## Initialisation

Initialise the weights and bias like

```python
import numpy as np

x = np.array([[0,0],[0,1],[1,0],[1,1]])
y = np.array([xor(*i) for i in x])

# I explicitly name these for clarity
n_neurons_input, n_neurons_hidden, n_neurons_output = 2, 2, 1

# if this confuses you draw out all the possible wasy to connect the
# first two neurons remembering that we have only connections between layers
# and not between neurons on the same layer
w_hidden = np.random.random(size=(n_neurons_input, n_neurons_hidden))
b_hidden = np.random.random(size=(1, n_neurons_hidden))

w_output = np.random.random(size=(n_neurons_hidden, n_neurons_output))
b_output = np.random.random(size=(1, n_neurons_output))
```
## Forward Propagation

We now have a neural network (albeit a lousey one!) that can be used to make a prediction. To make a prediction we must cross multiply all the weights with the inputs of each respective layer, summing the result and adding bias to the sum.

The action of cross multiplying all combinations of two vectors and summing the result is just the dot product, so that a single forward propagation is given as

```python
def sigmoid(x):
    return 1/(1 + np.exp(-x))

y_hidden = sigmoid(np.dot(x, w_hidden) + b_hidden)
y_output = sigmoid(np.dot(y_hidden, w_output) + b_output)
```

where `y_output` is now our estimation of the function from the neural network.

## Back propagation

The back propagation is then done

```python
def sigmoid_derivative(sigmoid_result):
    return sigmoid_result * (1 - sigmoid_result)

grad_output = (- y + y_output) * sigmoid_derivative(y_output)
grad_hidden = derror_dwo.dot(w_output.T) * sigmoid_derivative(y_hidden)
```

The updates require a little explanation of the cheats I did before in the theory section. The function, $a_o = w_o \cdot y_h + b_o$ is actually a vector equation so explicitly, $a_o = y_h^T w_o + b_o$ and similarly, $a_h = x^T w_h + b_h$ which gives some intuition why we need to do the transposes below

```python
w_output -= alpha * y_hidden.T.dot(grad_output)
w_hidden -= alpha * x.T.dot(grad_hidden)
```

The biases we recall had a derivative of 1. In reality this is the identity, or a vector of 1s so it reduces to a simple sum of all elements in the vector

```python
b_output -= alpha * np.sum(grad_output, axis=0, keepdims=True)
b_hidden -= alpha * np.sum(grad_hidden,axis=0, keepdims=True)
```

TBC...


# References

- https://towardsdatascience.com/implementing-the-xor-gate-using-backpropagation-in-neural-networks-c1f255b4f20d
- https://medium.com/@lucaspereira0612/solving-xor-with-a-single-perceptron-34539f395182
- https://blog.abhranil.net/2015/03/03/training-neural-networks-with-genetic-algorithms/