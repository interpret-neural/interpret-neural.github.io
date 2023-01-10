# On the Interpretability of Hyperparameters

By Joachim Wagner

Table of contents:

1. TOC
{:toc}

In this post, common hyperparameters in neural network training are
discussed from the perspective of interpretability.


## Overview

Since interpretability tries to help us better understand our models
and since hyperparameters influence the performance of our models, it
is important to ask to what extent hyperparameters are interpretable
and what can be done to improve their interpretability.
In the following, I will reflect on properties of common
hyperparameters with a focus on how they influence model performance.

I consider all configuration choices made for the learning process to
be hyperparameters.
Only a small subset of these choices are routinely listed as
hyperparameters.
Excluded are ordinary parameters that are set during each training run,
e.g. the weights and biases of a neural network.
Still, the two are not independent as there usually is a hyperparameter
search through which the outcome of the learning feeds back on the
choice of hyperparameters.


## Network Configuration

Within a given network architecture, a specific configuration can often
be described with a fixed length vector of number, e.g. specifying the
number of layers, the maximum input sequence length and the dimensions
of vectors.
These hyperparameters determine the total number of parameters and
therefore the capacity for storing information.
Some of these hyperparameters can be small natural numbers,
discouraging deviations from the values that worked well in previous
work as a step up or down can be expected to have large and probably
detrimental effects. 


## Training Configuration

### Initialisation

Usually the parameters of a neural network are randomly initialised.
Looking closer at what the initialisation entails, there are
hyperparameters here as one must specify from what distribution the
random values are sampled.
In the simplest case, a single uniform distribution is used throughout,
requiring to specify the interval from which the values are sampled.
While there are well-established best practices for initialisation, it
is hard to tell how much one can deviate from the recommendations
without harming performance and what the chances are to find
distributions that work better than the best practice.
Usually this hyperparameter is not explored and its contribution to the
model's performance is unknown.


### Optimisation

Rather than using a fixed learning rate or a simple parameterised
learning rate schedule, an adaptive optimiser is often used that has
its own parameters (3 in case of Adam).
While adjusting the learning rate automatically for each parameter has
clear advantages, it also increases complexity and obscures how the
hyperparameters related to the optimiser affect the learning process
and the resulting model.

With pre-trained models, one often sees different learning rates
applied to the pre-trained part of the neural network and the freshly
initialised "head".
Parts of the network can be frozen completely, such as in
[Houlsby et al.](https://arxiv.org/abs/1902.00751)'s
Adapters, or for a limited number of epochs.
These choices can be encoded as hyperparameters but typically only
binary choices are explored.
Binary choices, however, provide only limited insight as to how a
parameter behaves.

The optimisation process is guided by a loss function.
The choice of loss function is usually motivated theoretically, e.g.
cross entropy loss for multi-class classification.
If an alternative loss function is considered the choice becomes a
categorical hyperparameter.

Other hyperparameters related to optimisation describe the
training-development split (size, randomisation, stratification,
deduplication), the batch size, how batches are assembled, the duration
of training and dropout probabilities.
(The latter can also be seen to be part of the network configuration.)
While these hyperparameters are easy to understand, their effect on
model performance is hard to predict and intuitions about the best
choice can be misleading, requiring experimentation.


## Model Selection

Training usually proceeds in epochs, each epoch consuming all training
items once and only once and monitoring progress on development data at
the end of each epoch but it is also common to create checkpoints more
often and monitor progress on a subset of these checkpoints, e.g. every
10th checkpoint.
Hyperparameters control when training stops and how the final model is
chosen from the available checkpoints.
Usually there is a minimum number of epochs, a patience that pushes out
the number of epochs if the best model is recent and a model selection
criterion.
To interpret the number of epochs and patience, one needs to see them
in relation to the variation of performance between epochs and the
general trend of the performance near the end of training.


## Random Seeds

In his preprint
"[We need to talk about random seeds](https://arxiv.org/abs/2210.13393)",
Bethard argues that the initialisation of pseudo random number
generators (PRNG) from which the random numbers needed for the learning
process are taken should be considered a hyperparameter and treated
equally.
Indeed, the random seed is a configuration parameter that influences
the learning process and hence meets our above definition of a
hyperparameter.
Uses of PRNGs include, for example, the initialisation of parameters,
the shuffling or sampling of training data, data augmentation and
dropout.
For many PRNGs, the sequence of numbers produced changes totally
regardless of whether the random seed is changed a lot or a little.
The performance of a model cannot be predicted from the random seed, at
least not for a new network configuration.
(If a network worked well with a seed for task A it may also work well
for task B, though this is not guaranteed.)
This makes the random seed hyperparameter fully opaque and limits
interpretation to statistics.

Furthermore, the effect of the random seed is tightly coupled to the
implementation, e.g. the order in which parameters are initialised and
the choice of PRNG.
If a change in the model configuration changes how many random numbers
are required to initialise one component, this will affect the
initialisation of remaining components
(unless each component uses its own PRNG seeded from a combination of
the run's main seed and a component identifier).


## Discussion

Hyperparameters vary in how transparent they are and often are not
explored as their configuration is copied from previous work or
software.
To answer empirically how a hyperparameter choice affects model
performance, one has to train many models.
This limits how many practitioners can afford to explore the behaviour
of hyperparameters and to build an intuition about them.
While hyperparameters affect the overall learning process and not
individual predictions, they can still have systematic effects relevant
to individual predictions, e.g. how well minority classes are predicted
via hyperparameters controlling the loss function and/or sampling of
training data.

Overall performance may not be sufficient to measure influence of
hyperparameters.
Two different choices for hyperparameters may lead to similar overall
performance but this does not mean that predictions must be similar.
The main types of errors may change, causing a low overlap in errors
between two models.
(Such differences are exploited in ensemble methods.)
The number of changes in the predictions may be a better measure of the
effect of a particular change in hyperparameters.
Still, hyperparameters may have other effects, e.g.
[Bansal et al.](https://openaccess.thecvf.com/content_CVPR_2020/html/Bansal_SAM_The_Sensitivity_of_Attribution_Methods_to_Hyperparameters_CVPR_2020_paper.html)
show that explanations for individual predictions are sensitive to
hyperparameter choices. 


## References

Naman Bansal, Chirag Agarwal and Anh Nguyen (2020) SAM: The Sensitivity
of Attribution Methods to Hyperparameters. In Proceedings of the
IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR),
2020, pp. 8673-8683.

Steven Bethard (2022) We need to talk about random seeds.
https://arxiv.org/abs/2210.13393

Neil Houlsby, Andrei Giurgiu, Stanislaw Jastrzebski, Bruna Morrone,
Quentin De Laroussilhe, Andrea Gesmundo, Mona Attariyan, Sylvain Gelly
(2019) Parameter-Efficient Transfer Learning for NLP. In Proceedings of
the 36th International Conference on Machine Learning,
PMLR 97:2790-2799.
