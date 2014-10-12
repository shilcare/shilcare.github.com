---
layout: post
title: Energy Based Model Notes
comment: true
categories: ml

---

Energy based model学习笔记。

## Softmax


When soloving classification problems, why do we not use squared error measure as the loss function? Because squared error measure has some drawbacks:

- If the desired output is 1 and the actual output is $$1e-9$$ then there is almost no gradient for a logistic unit to fix up the error as the slope is nearly horizontal, it will take a very very long time to change its weights, even though it's making a very big error.  
Suppose that we use a squared error cost function with a logistic unit, $$E=\frac{1}{2}(y-t)^2$$, where $$y=\sigma(z)=\frac{1}{1+\exp(-z)}$$, then $$\frac{dE}{dz}=y(1-y)(y-t)$$. When $$y$$ is close to 0 or 1, the derivatives tends to zero, so if we update the weights using gradient descent, they will almost not change at all.
- If we are trying to assign probabilities to *mutually exclusive* class labels, we know that the output should sum to 1, but we are depriving the network of this knowledge.  

The softmax function was proposed to solve the second problem, it forces the outputs of the neural net to represent a probability distribution across discrete class labels.  
![Softmax group]({{ site.url }}/assets/images/softmax.jpeg)  
The output units in a softmax group each receives some total input they've accumulated from the layer below, denote by $$z_i$$ for the i-th unit, and that's called the *logit*, then the output of the i-th unit:

$$y_i=\frac{e^{z_i}}{\sum \limits_{j\in group}e^{z_j}}$$

The partial derivative is in a form very like to that we see of the logistic unit:

$$\frac{\partial y_i}{\partial z_i} = y_i(1-y_i)$$

Using a *cross entropy* cost function $$E=-t\log(y)-(1-t)\log(1-y)$$ will fix this problem because then $$\frac{dE}{dz}=y-t$$

