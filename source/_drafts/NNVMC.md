---
title: Neural Network Quantum Monte Carlo
tags: computational methods
license: BY-SA
---

Replace traditional ansatz with neural network ones, in variation quantum Monte Carlo.
Also known as NQS (neural quantum states), NN-VMC, ML-VMC... 
<!-- more -->

## MLP
$\mathbf{r} → \underbrace{\mathbf{W\;r+b} → f()}_{\rm MLP\,layer} → ... → \Psi$

The weights $\mathbf{W,b}$ is trainable parameters.

when the activate function is chose as radial basis functions, the network is also categorized as RBF. 
> Radial basis function, e.g.:
$x → e^{-\#|x-\#|}, e^{-\# |x-\#|^2}, \sqrt{|x-\#|^2+\#^2}, (|x-\#|^2+\#^2)^{-1/2}$


## RBM
RBMs approximates the **direct interacting system** between physical nodes with a system interacting with **hidden nodes**.
>To some extent, it's just a single-layer MLP, with $\cosh$ as activate function.

The network is set as a "Ising" system, with a hidden layer $\mathbf{h}$.
The probability of an input $\mathbf{\sigma}$ obeys the Boltzmann distribution.
All the visible "spins" $\sigma_i$ and hidden "spins" $h_j$ are 0 or 1.
$Z = \sum_\mathbf{\sigma,h} e^{-E} = \sum_\mathbf{\sigma,h} e^{- W_{ij} \sigma_i h_j - b_i \sigma_i - c_j h_j } = \sum_\mathbf{\sigma} \cosh({\mathbf{ W_{ij} \sigma_i - c_j}}) e^{-b_i \sigma_i}$ 
where $\mathbf{W,c}$ is the trainable parameters.
The probabiltiy is $p(\sigma) = \frac{1}{Z} \sum_h e^{-E(\sigma,h)} \propto \cosh({\mathbf{ W_{ij} \sigma_i - c_j}}) e^{-b_i \sigma_i}$

For inferrence run, when the parameters are fixed, 
an input of $\sigma$ gives a probability. 

The wavefunction is complex, 
so there are two common choices of constructing the network, 
i) use complex network: allow all paramters to be complex, 
ii) use two networks for real and imaginary part of $\Psi$ respectively.


## GNN
In lattice models, there are well defined "neighbours".

> In continuous space, there're also message-passing networks.
Keeps a message flow other than the original one- and two-electron streams in FermiNet. They seems to not restrict the message to be local.


## backflow with determinants
FermiNet, PauliNet, DeepSolid

In mean-field methos, the wavefunction is





## Devils in the details

### optimizers: Adam, AdamW, SR, minSR, K-FAC, 

### the envelope 

### MCMC sampling problem
