---
layout: post
title: Graph Neural Networks and Wind Farms
subtitle: Deep Neural Networks with Relational Inductive Biases for Wake Modeling
bigimg: /img/wakes.jpg
tags: [wake effects, deep learning, wind farms, Graph Neural Networks]
---
# Wake effects
As discussed in my [other post](https://mylonasc.github.io/2019-01-17-farmVAE/) wake effects is an important consideration when designing windfarms.

It is not straight-forward to generalize the feed-forward VAE work to be able to deal with arbitrary farm configurations.
Moreover, ideally we would like to be able to learn wake effects as a function of micro-climate conditions and/or different turbines.
In some sense the turbines are *"entities"* and their interactions are *"relations"*. 

Following a great paper from [DeepMind](https://deepmind.com/) which unified several approaches to computation on graphs( [Relational inductive biases, deep learning, and graph networks](https://arxiv.org/pdf/1806.01261.pdf) ) and using the open-sourced GraphNets library they released, I modeled with the so called **GraphNets** a large windfarm to see if wake effects are effectively captured.

## Farms and Graphs
In the context of this work,
* Wind turbines are nodes
* Turbine-turbine and intra-turbine interactions are edges
* global parameters are weather conditions.

In an initial attempt, a manual pre-selection of properties and variables describing these graph parameters were made. The data from a wind-farm of 111 turbines was used for training a graph neural network (full GraphNet). The farm I studied spans 20km by 10km radius. Therefore some data cherry picking was necessary in order for the variation of operational and global environmental parameters along the whole farm to be small. In the future I want to see if I can capture the underlying wind field as a latent effect. That could be possible through modeling the wind field, for instance, on a grid or a mesh 
superimposed on the farm, with dynamics learned directly from data. The work on [Attentive Neural Processes](https://arxiv.org/abs/1901.05761), which are loosely related 
to Gaussian Processes is related to this idea. Computation on unstructured meshes can be  
facilitated with GraphNets ([Learning Mesh-Based Simulation with Graph Networks](https://arxiv.org/abs/2010.03409) and in particular [Graph Element Networks](https://arxiv.org/abs/1904.09019) are contain some good ideas on the details on how to achieve this).

The first results seemed quite promising
![Actual and predicted farm state (power and turbine orientation) for all turbines, given only global turbulence, wind speed and wind orientation. The arrows represent the wind inflow orientation at each turbine.](/img/power_on_farm_lowTi.png)

![same as previous plot for higher windspeed](/img/power_on_farm_lowTi_HighWsp.png)

Although this part may not seem very convincing, one needs to keep in 
mind that the global operational characteristics are parameterized by 
3 values and we ask of the network to predict how much all turbines 
in a 20km by 10km area produce! The weather conditions are of 
course not expected to be the same all over the farm and at 
the moment I take no measures to take that into account. The 
only parameters entering the network are special features encoding 
the turbine relative position, and the turbine rated power. 
I also use the encode/process/decode approach used in the DeepMind paper and small ReLU networks.
The GraphNets were fast to train and converged reliably.

### Generalization to unseen farm configurations
I find GraphNets exciting, also due to the potential of generalizing to different farm configurations and learning physical laws in a data-driven manner. 
The farm-level effects (wakes) are not trivial to model accurately with explicit computational models. 
High-fidelity models need dedicated super-computers and medium fidelity, make relatively coarse empirical approximations (Dynamic Wake Meandering method) which may sacrifice accuracy in the representation of the actual aero-dynamic loads. The goal here would be to be able to learn wake models **directly from operational data** and turbine properties of several farms and generalize in arbitrary configurations. The main direct applications of such models are:
* prediction of farm performance, 
* farm-level control, and
* farm-layout optimization

Here are some preliminary results on the trained graph-network on a grid-layout farm. Keep in mind that the network was trained on a completely different layout.
![Wake effects on a 5x5 grid layout. The wake effects are correctly larger in the internal part of the farm.](/img/fictitious5x5.png)


Note the lighter region denoting higher power production. The upwind turbines (the ones on the sides that are facing the wind as it reaches the farm) produce more. The interesting 
feature is that this was learned directly by the model which had only seen the large 111-turbine farm. The interesting feature of GraphNet is that one may generalize to larger unseen 
configurations with no extra training cost. A larger 10x10 farm is shown in the following
![Wake effects on a 10x10 grid layout farm.](/img/fictitious10x10.png)

and a 20x20 farm:
![Wake effects on a 20x20 grid layout farm.](/img/fictitious20x20.png)

the same effect persists which is encouraging as to the capacity of GraphNets to capture physical laws.

### Conclusion/Future work
As a conclusion, graph neural networks show great potential as a modeling tool for wind farms. Further interesting extensions would include a more careful account of 
uncertainty and [Bayesian deep neural networks](https://arxiv.org/abs/1506.02557) for learning the interaction functions with the associated uncertainty. (**update:** In [Bayesian graph neural networks for strain-based crack localization](https://arxiv.org/abs/2012.06791) I battle tested this idea for the first time and it seemed to yield satisfactory results.).


