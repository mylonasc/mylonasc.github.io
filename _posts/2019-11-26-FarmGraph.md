---
layout: post
title: Graph Neural Networks and Wind Farms
subtitle: Deep Neural Networks with Relational Inductive Biases for Wake Modeling
bigimg: /img/wakes.jpg
tags: [wake effects, deep learning, wind farms, Graph Neural Networks]
---
# Wake effects
I explain what are wake effects and why they are important for wind-farm performance monitoring and remaining life assessment in my [other post](https://mylonasc.github.io/2019-01-17-farmVAE/). It is hard to generalize the previous work to be able to deal with arbitrary farm configurations and process simultaneously data from different farms. For some time now (namely since Feb. 2019) I have been thinking of treating the interactions of wind turbines in a farm as a graph.

In 2018 [DeepMind](https://deepmind.com/) has published a paper unifying several approaches to computation on graphs: [Relational inductive biases, deep learning, and graph networks](https://arxiv.org/pdf/1806.01261.pdf). 

In my opinion this is a landmark paper. That is, because it provides a language to think about graph neural networks. The paper shows 3 applications on graph neural networks. Namely, 
* sorting arbitrary list 
* finding the shortest path in a graph
* predicting the dynamic evolution of a chain

These tasks are supposed to be learned by example.

## Farms and Graphs
In my context,
* Wind turbines are nodes
* Turbine-turbine and intra-turbine interactions are edges
* global parameters are weather conditions.

In an initial attempt, a manual pre-selection of properties and variables describing these graph parameters were made. The data from a wind-farm of 111 turbines was used for training a graph neural network. The library released with the aforementioned paper was used. The farm I studied spans 20km by 10km radius. Therefore some data cherry picking was necessary in order for the variation of operational and global environmental parameters along the whole farm to be small (more details on the paper to come!). 

The results seemed quite promising!
![Actual and predicted farm state (power and turbine orientation) for all turbines, given only global turbulence, wind speed and wind orientation. The arrows represent the wind inflow orientation at each turbine.](/img/power_on_farm_lowTi.png)

![same as previous plot for higher windspeed](/img/power_on_farm_lowTi_HighWsp.png)

This part may not seem very convincing. However, keep in 
mind that the global operational characteristics are parameterized by 
3 values and we ask of the network to predict how much all turbines 
in a 20km by 10km area produce! The weather conditions are of 
course not expected to be the same all over the farm and at 
the moment I take no measures to take that into account! The 
only parameters entering the network are special features encoding 
the turbine relative position, and the turbine rated power. 
I also use the encode/process/decode approach advocated in the DeepMind paper.

### Generalization to unseen farm configurations
What I find exciting about graph neural networks, is the potential of generalizing to different farm configurations and learning physical laws in a data-driven manner. General Artificial Intelligence, the reason why they were invented, for me is a *nice-to-have* side-effect when it happens! The farm-level effects (wakes) are not trivial to model accurately with explicit computational models. High-fidelity models need dedicated super-computers and medium fidelity, make relatively coarse empirical approximations (Dynamic Wake Meandering method) which may sacrifice accuracy in the representation of the actual aero-dynamic loads. The goal here would be to be able to learn wake models directly from operational data and turbine properties of several farms and generalize in arbitrary configurations. The main direct applications of such models are:
* prediction of farm performance, 
* farm-level control, and
* farm-layout optimization

Here are some preliminary results on the trained graph-network on a grid-layout farm. Keep in mind that the network was trained on a completely different layout.
![Wake effects on a 5x5 grid layout. The wake effects are correctly larger in the internal part of the farm.](/img/fictitious5x5.png)


Going bigger...
![Wake effects on a 10x10 grid layout farm.](/img/fictitious10x10.png)

and bigger...
![Wake effects on a 10x10 grid layout farm.](/img/fictitious10x10.png)

### Conclusion/Future work
As a conclusion, apparently graph neural networks show great potential as a modeling tool for wind farms. Although this idea has been [explored before](https://www.sciencedirect.com/science/article/abs/pii/S0360544219315555), I think it may have a lot to offer. Especially if combined with other new developments... 


