---
layout: post
title: Graph Neural Networks and Wind Farms
subtitle: Deep Neural Networks with Relational Inductive Biases for Wake Modeling
bigimg: /img/wakes.jpg
tags: [wake effects, deep learning, wind farms, Graph Neural Networks, PhD]
---
# Wake effects
As discussed in my [other post](https://mylonasc.github.io/2019-01-17-farmVAE/), wake effects are an important consideration when designing wind farms.

It is not straightforward to generalize the feed-forward VAE work to be able to deal with arbitrary farm configurations.
Moreover, ideally, we would like to be able to learn wake effects as a function of micro-climate conditions and different turbines.
In some sense, the turbines are *"entities,"* and their interactions are *"relations"*. 

Following a great review and position paper from [DeepMind](https://deepmind.com/) which unified several approaches to computation on graphs( [Relational inductive biases, deep learning, and graph networks](https://arxiv.org/pdf/1806.01261.pdf) ) and using the open-sourced GraphNets library they released, I modeled with the so-called **GraphNets** a large windfarm to see if wake effects are effectively captured.

## Farms and Graphs
In the context of this work,
* Wind turbines are nodes
* Turbine-turbine and intra-turbine interactions are edges
* global parameters are weather conditions.

In an initial attempt, a manual pre-selection of properties and variables describing these graph parameters was made. The data from a wind farm of 111 turbines were used for training a graph neural network (full GraphNet). The farm I studied spans a 20km by 10km radius. Therefore some data cherry-picking was necessary in order for the variation of operational and global environmental parameters along the whole farm to be small. In the future, I want to see if I can capture the underlying wind field as a latent effect. That could be possible through modeling the wind field, for instance, on a grid or a mesh superimposed on the farm, with dynamics learned directly from data. Or having some spatiotemporal GP, probably through random Fourier features as a node-to-global function. The work on [Attentive Neural Processes](https://arxiv.org/abs/1901.05761), which are loosely related to Gaussian Processes, is related to this idea. Computation on unstructured meshes can be facilitated with GraphNets ([Learning Mesh-Based Simulation with Graph Networks](https://arxiv.org/abs/2010.03409) and in particular [Graph Element Networks](https://arxiv.org/abs/1904.09019) contains some good ideas on the details on how to achieve this).
**update - June 2021**: I did my version of neural processes to accommodate edge latent variables and conditioning - here's the preprint of the paper: [Relational VAE](https://arxiv.org/abs/2106.16049). 

The first results seemed quite promising.

![Actual and predicted farm state (power and turbine orientation) for all turbines, given only global turbulence, wind speed, and wind orientation. The arrows represent the wind inflow orientation at each turbine.](/img/power_on_farm_lowTi.png)

![same as the previous plot for higher windspeed](/img/power_on_farm_lowTi_HighWsp.png)

Although this part may not seem very convincing, one needs to keep in mind that the global operational characteristics are parameterized by 3 values, and we ask the network to predict the operational statistics of all turbines in a 20km by 10km area will operate. The weather conditions are, of course, not expected to be the same all over the farm, and at the moment, I take no measures to take that into account. The only parameters entering the network are special features encoding the turbine relative position and the turbine rated power. 
The encode/process/decode approach was used is n the DeepMind paper and small ReLU networks. The GraphNets were fast to train and converged reliably.

### Generalization to unseen farm configurations
I find GraphNets exciting, also due to the potential of generalizing to different farm configurations and learning physical laws in a data-driven manner. 
The farm-level effects (wakes) are not trivial to model accurately with explicit computational models. 
High-fidelity models need dedicated super-computers and medium-fidelity, make relatively coarse empirical approximations (Dynamic Wake Meandering method) which may sacrifice accuracy in the representation of the actual aerodynamic loads. The goal here would be to be able to learn wake models **directly from operational data** and turbine properties of several farms and generalize them in arbitrary configurations. The main direct applications of such models are:
* prediction of farm performance, 
* farm-level control, and
* farm-layout optimization

Here are some preliminary results on the trained graph network on a grid-layout farm. Keep in mind that the network was trained on a completely different layout.
![Wake effects on a 5x5 grid layout. The wake effects are correctly larger in the internal part of the farm.](/img/fictitious5x5.png)


Note that the lighter region is denoting higher power production. The upwind turbines (the ones on the sides that are facing the wind as it reaches the farm) produce more. This was learned directly by the model, which had only seen the large 111-turbine farm. The interesting feature of GraphNet is that one may generalize to larger unseen configurations with no extra training cost. A larger 10x10 farm is shown in the following, as well as a 20x20 farm.

![Wake effects on a 10x10 grid layout farm.](/img/fictitious10x10.png)

![Wake effects on a 20x20 grid layout farm.](/img/fictitious20x20.png)

The same effect persists, which is encouraging as to the capacity of GraphNets to capture physical laws. Finally, it is known that after some distance, the wake-related production deficits are not observed due to kinetic energy transferred from wind over the farm. This is what is probably what is observed in the final plot, although more examples should be investigated.

### Conclusion/Future work
In conclusion, graph neural networks show great potential as a modeling tool for wind farms (and many other fields of engineering).
Further interesting extensions would include a more careful account of uncertainty and [Bayesian deep neural networks](https://arxiv.org/abs/1506.02557) for learning the interaction functions with the associated uncertainty. (**update:** In [Bayesian graph neural networks for strain-based crack localization](https://arxiv.org/abs/2012.06791), I battle-tested this idea for the first time, and it yields satisfactory results.).

<h3></h3><!-- Start BawkBox Code--><script data-sil-id="6035570e3c0d090013685d5b">var loadWidget = function() { var d = document, w = window, l = window.location,p = l.protocol == "file:" ? "http://" : "//"; if (!w.WS) w.WS = {}; c = w.WS; var m=function(t, o){ var e = d.getElementsByTagName("script"); e=e[e.length-1]; var n = d.createElement(t); if (t=="script") {n.async=true;} for (k in o) n[k] = o[k]; e.parentNode.insertBefore(n, e)}; m("script", { src: p + "bawkbox.com/widget/like-dislike/6035570e3c0d090013685d5b?page=" +encodeURIComponent(l+''), type: 'text/javascript' }); c.load_net = m; }; if(window.Squarespace){ document.addEventListener('DOMContentLoaded', loadWidget); setTimeOut(function(){ document.addEventListener('DOMContentLoaded', loadWidget); }, 3000) } else { loadWidget() } </script><div class="sil-widget-like-dislike sil-widget" id="sil-widget-6035570e3c0d090013685d5b"><a href="//bawkbox.com/install/like-dislike">Like Dislike Button</a></div><!-- End BawkBox Code-->
