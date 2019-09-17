---
title: Paper_Notes_About_Recommendation_in_AAAI19
date: 2019-09-10 15:53:58
tags: [Recommender Systems, Knowledge graph, Attention Mechanism]
---

# Overview

## Explainability

Recently, the recommendation community has reached a consensus that accuracy can only be used to partially evaluate a system. Explainability of the model, which is the ability to provide explanations for why an item is recommended, is considered equally important.

Papers like follows:

1. [Explainable Reasoning over Knowledge Graphs for Recommendation](#explainable-reasoning-over-knowledge-graphs-for-recommendation): Use knowledge graph(KG) info for expalainability. Select the most related path in KG as the explanation.
2. [Explainable Recommendation Through Attentive Multi-View Learning](#Explainable Recommendation Through Attentive Multi-View Learning): Construct a hierarchical (explicit) feature (e.g. meat, seafood and so on) tree for explainability. Select the most related features as explanation.
3. [Dynamic Explainable Recommendation based on Neural Attentive Models](#Dynamic Explainable Recommendation based on Neural Attentive Models): Generate dynamic explanations at different time for a user by using GRU. An explanation is the most related sentence in the review of the recommended item.

## Sequential Recommendation

Different from traditional MF that treats users and items equally, the methods model the interactions between users and items as a number of historical interaction sequences of users.  Then focusing on the sequential interactions of users, some methods was proposed to analysis the phenomenon like long-term interests, drifted interests, repeat consumption and some other task-specific aspects. The tasks include next item (basket) prediction (like POI), session recommendation. 

1. [A Repeat Aware Neural Recommendation Machine for Session-based Recommendation](#A Repeat Aware Neural Recommendation Machine for Session-based Recommendation): Emphasize the importance of repeat consumption.
2. [DAN: Deep Attention Neural Network for News Recommendation](#DAN: Deep Attention Neural Network for News Recommendation): Do a specific task with a complex combination of popular networks. The emphasized point is still the order info that has been studied for general recommendation over and over again.
3. [Session-based Recommendation with Graph Neural Networks](#Session-based Recommendation with Graph Neural Networks): Model item transitions in sessions as a Graph.
4. [Hierarchical Context enabled Recurrent Neural Network for Recommendation](#Hierarchical Context enabled Recurrent Neural Network for Recommendation): Emphasize global long-term interests, local sub-sequence interests and the temporary interests of each transition in a sequence.
5. [A Spatio-Temporal Gated Network for Next POI Recommendation](#A Spatio-Temporal Gated Network for Next POI Recommendation): Emphasize the influence of spatio-temporal intervals between check-ins for POI recommendation.

## Domain Combination

Use the complete idea in one domain to model and solve the problem in another domain.

Papers like follows:

1. [From Zero-Shot Learning to Cold-Start Recommendation](#From Zero-Shot Learning to Cold-Start Recommendation): Use ZSL method to solve CSR problem
2. [Meta Learning for Image Captioning](#Meta Learning for Image Captioning)

## Others

1. [A Collaborative Ranking Method for Content Based Recommendation](#A Collaborative Ranking Method for Content Based Recommendation): Follow a traditional line that generates item embedding from auxiliary info and then combine it with MF like $u_i^T(v_j + embedding(doc_j))$
2. From Recommendation Systems to Facility Location Games: An interest aspect for recommendation problem. The authors design a Mediator (for service providers) to direct what the content providers(e.g. blog/video publishers) should provide to achieve overall higher social welfare while intervene as little as possible. The introduction of method is omitted for it is more related to social science and is a bit complex.

<!-- more -->

# Session-based Recommendation with Graph Neural Networks 

Problem: Session Recommendation

Gap: Transitions among distant items are often overlooked by previous methods. 

Method: Use Graph to model item relation in sessions and utilize Graph Neural Networks to capturing transitions of items and generate accurate item embedding vectors. For example, as follows, the three sessions are constructed as a graph.

![Session graph](img/aaai19/session-graph.PNG)

# A Collaborative Ranking Method for Content Based Recommendation 

Problem: A traditional line that uses auxiliary information to enrich item embedding based on MF. The general score function for user $i$, item $j$ and auxiliary info $doc_j$ is as followings:
$$
f(i,j,doc_j) = p_i^T(q_j + g(doc_j))
$$
where $g(.)$ is the encoding function.

Gap: For function $g(.)$, some methods ignore the order of words, while others fail to identify the high leveled topic info.

Method: An encoding function based on GRU(capture order) and multi-head attention mechanism(model topic info).

![CAMO](img/aaai19/camo.PNG)

A Nice Attention Introduction: [https://lilianweng.github.io/lil-log/2018/06/24/attention-attention.html](https://lilianweng.github.io/lil-log/2018/06/24/attention-attention.html)

# Explainable Reasoning over Knowledge Graphs for Recommendation

Problem: recommendation with explainability, using KG specifically.

Gap: The connectivity information in KG can help to endow recommender systems the ability of reasoning and explainability. However, some traditional methods(use meta-paths) need domain knowledge to predefine meta-paths, while others(embedding based, like using TransE) are less explainable.

Method:

1. Model user-item interaction $(u, i)$ as KG triplet relation $(u, interact, i)$.

2. For each $(u,i)$, select all paths from user node $u$ to item node $i$, denoted as $P(u,i) = \{p_1, p_2, ..., p_K\}$

3. The score function from user $u$ to item $i$ is modeled as:
   $$
   \hat{y}_{ui} = f_{\Theta}(u,i|P(u,i))
   $$

4. Use LSTM to model the preference of each path and use a weighted pooling layer(not attention) to calculate the final preference and estimate the importance of each path.

   ![kg-lstm](img/aaai19/kg-lstm.PNG)

Weighted Pooling Layer:

For each preference $s_k$ outputted from the LSTM layer, the weighted pooling layer is defined as follows:
$$
g(s_1, s_2, ..., s_K) = log[\sum_{k=1}^K \exp (\frac{s_K}{\gamma})]
$$
where $\gamma$ is a hyper-parameter to control each exponential weight and the path importance is estimated from the gradient on each path preference:
$$
\frac{\partial g}{\partial s_k} = \frac{\exp (s_k/ \gamma)}{\gamma \sum_{k^{'}} \exp (s_{k^{'}}/\gamma)}
$$
Finally,  the score function is 
$$
y_{ui} = \sigma(g(s_1,s_2,...,s_K))
$$

# From Zero-Shot Learning to Cold-Start Recommendation 

Problem: Cold-start Recommendation(CSR)

Gap: no gap but a novel view in light of Zero-Shot Learning(ZCL). Actually, I am not used to this kind of writing pattern.

Challenge to formulate CSR as a ZSL problem: 

1. Domain shift problem: the data distribution is not same as in ZSL
2. Data sparsity: the interactions in recommendation domain is very sparse.
3. Efficiency. 

Method: Use a Linear Low-rank Autoencoder to learn a transform matrix $W$ that transfer user behavior matrix $X$ into user attributes matrix $S$ as follows:
$$
\min_{W}|| X - W^TWX||_F^2 + \beta rank(W),  s.t. WX = S
$$
Then, for new users, we can estimate their potential user behavior matrix $X_{new}$ as
$$
X_{new} = W^TS_{new}
$$
My concern is how the autoencoder or low-rank constraint solve the data sparsity problem in Recommendation domain. In ZSL, each $X$ is an image with enough info. However, in RS, each $X$ is an interaction matrix which is very sparse and in total, there is only one large matrix with the user number as row number and item number as column number as train data. Compare to ZSL, the train dataset is also very small.

A similar low-rank Autoencoder for ZSL in IJCAI'18: [Zero Shot Learning via Low-rank Embedded Semantic AutoEncoder](https://www.ijcai.org/proceedings/2018/0345.pdf)

# Meta Learning for Image Captioning

Problem: Image caption

Gap: reward hacking problem in RL. Shown in the following graph, the reward (digit next to the caption) of caption generated by RL is even higher than the ground truth (GT) caption while the caption quality is actually not.

![Reward hacking in image caption](img/aaai19/reward-hacking.PNG)

Method: Utilize the power of meta-learning to ensure the correctness and distinctiveness of the generated captions(by RL reward) and optimize the evaluation metrics(by maximizing the likelihood of the ground truth caption). Shown in the following graph, the meta-learning objective(marked in green) could the learn $\theta$ that is optimal to adapt both tasks while just adding RL reward $L_1(\theta)$ with maximize likelihood target $L_2(\theta)$takes a gradient in between (marked in brown).

![meta loss vs aggregation](img/aaai19/meta-loss-vs-aggregation.PNG)

The idea behind this work may suggest that we should use meta-learning optimization method to substitute trivial aggregation of different losses in all kinds of works if there is the similar hacking problem in the corresponding domains.

# Explainable Recommendation Through Attentive Multi-View Learning

Problem: recommendation with explainability

Gap: On one hand, some deep learning-based methods lack of explainability. On other hand, other shallow models lack of an effective mechanism to model high-level explicit features which limits their accuracy. What's more they can't identify hierarchical interests. As shown in the following graph, previous works can not identify whether a user is interested in lower-level features such as $\textit{shrimp}$ or higher-level features like $\textit{seafood}$.

![hierarchical interests](img/aaai19/hierarchical-interests.PNG)

Method: feed the model with the hierarchical features information. An overview is as follows.

1. Build an explicit feature hierarchy $\gamma$ with in total $L$ features and $H$ layers from the collected data like [Microsoft Concept Graph](https://concept.research.microsoft.com/ ). Pretrain with the graph to get the embedding $e_i$ of each feature node.
2. For a user $i$,  collect his explicit user-feature interest $x_i$ from his reviews. $x_{il}$ indicates the frequence of the feature $F_l$ being mentioned by the user.
3. propagate the feature-interest on the tree to get more proper interest $\hat{x_i}$ to let each interest on the feature node contain the interests on its sub-nodes. 
4. For items, build their features denoted as $\hat{y_j}$ from product introduction in a similar way. 
5. Attentive multi-view learning with the ratings and collected features as supervising info. Each user and item has an implicit feature and an explicit feature. Both features need fit ratings based on MF while the explicit features need also  fit collected features with a linear transform and L2 loss.

The overall process is as follows.

![](img/aaai19/multi-view-learning.PNG)

Then for explainability, the method utilize the feature hierarchy $\gamma$ and the trained model to select some important features as reason of recommendation.

Additionally, from the experimental results shown in the following figure, we can see SVD++ performs well with RMSE metric and even performs better than some deep models likes DeepCoNN and NARRE that using review info. The phenomenon should be discussed more detailly.

![](img/aaai19/multi-view-learning-exp.PNG)

# Dynamic Explainable Recommendation based on Neural Attentive Models

Problem: recommendation with explainability

Gap: Most previous works represent a user as a static latent vector($most$ is a good word). More importantly, *the explanations provided by these models are usually invariant*.

Method: Use GRU to model dynamic interest of users. Add a time gate to GRU to use the message of time interval. For example, a user tend to have similar preferences within a short time, while large time gap may have more opportunities to change user interest. For an item, use the hidden state $h_s$ of GRU at current time as the user's interest to select the most related sentence in the reviews as explanation by making use of attention mechanism. The general process is shown in following figure.

![](img/aaai19/dynamic-gru.PNG)

An example of explanation is shown as follows.

![](img/aaai19/dynamic-explanation-demo.PNG)

# DAN: Deep Attention Neural Network for News Recommendation 

Problem: News recommendation (next item recommendation)

Gap: They usually fail with the dynamic diversity of news and user’s interests, or ignore the importance of sequential information of user’s clicking selection. (This is obviously a false assertion for general recommendation) 

Method: Combination of CNN, RNN and Attention Mechanism. 

![](img/aaai19/news-rec.PNG)

As shown in the above figure, the process is summary as:

1. Use two CNN(PCNN) to extract embedding of news with title and profile parts.

2. Use LSTM and Attention to learn a user's embedding with a sequence of clicked news.

3. Score function is based on similarity between user embedding and candidate news embedding as follows. The $cosine$ function is calculated by the inner product of two vectors which is the same with MF but with different name.
   $$
   P = cosine(\hat{I}_t . I_t)
   $$

# A Repeat Aware Neural Recommendation Machine for Session-based Recommendation

Problem: session recommendation

Gap: propose a novel *repeat consumption* phenomenon where the same item is re-consumed repeatedly over time. The following table shows the repeat ratio in three public datasets.  However, no previous works have emphasized *repeat consumption* with neural networks. (Multi-armed bandit problem has been studied for a long time to solve the repeat-explore dilemma for general recommendations.)

![](img/aaai19/repeat-ratio.PNG)

Method: 

![](img/aaai19/repeatnet.PNG)

As shown in the above figure, the process is summarized as:

1. Use GRU to encode the session sequence of a user.
2. Use attention mechanism to merge all hidden states with the last hidden state as attention point. Then use a dense network to transform the merged vector into a two dimension vector with the first value as the probability of *repeat* and the other as the probability of *explore*
3. For repeat mode, decode the hidden states to calculate score for each item (new items get 0 point)
4. For explore mode, decode the hidden states with attention to calculate for each item (old items get 0 point)
5. Use the score and whether the item is new as supervising info to train model with a negative log-likelihood loss and a logistic loss.

# Hierarchical Context enabled Recurrent Neural Network for Recommendation 

Problem: Next-item recommendation

Gap: Emphasize long-term interests, local sub-sequence interests and temporary interests of each transition which are shown in the following graph while all these interests have only been partially reflected in most previous works.

![](img/aaai19/sequential-interest.PNG)

Method: (the authors assume each sequence should be longer than 10 in experiments)

Traditional LSTM process is shown as follows:

![](img/aaai19/hcrnn-lstm.PNG)

The authors propose to use cell state $c_t$ to present the local-sub sequence interests and use hidden state $h_t$ to present temporary transition interests. However, in traditional LSTM, $h_t$ are only directly dependent on $c_t$, there is little difference between them. Besides the long-term interests are neglected if $c_t$ is treated only as local interest. So, the author modify to LSTM to let $h_t$ more independent and add a memory network to keep long-term interests. The process is shown as 

![](img/aaai19/hcrnn-1.PNG)

where the parameter $\theta$ are trainable parameter $\theta$ for attention mechanism. Then in each transition, the influence of global long-term interests is considered.

The general process is shown in the following graph in which other tricks like bi-channel attention for prediction to focus on all interests are simply demonstrated.

![](img/aaai19/hcrnn-arch.PNG)

One way to demonstrate global interests is shown in the following graph from which we can see global context vector cover the most of the items.

![](img/aaai19/hcrnn-embedding.PNG)

# A Spatio-Temporal Gated Network for Next POI Recommendation 

Problem: POI recommendation (next item prediction)

Gap: Most previous recommendation methods don't consider both time intervals and geographical distances between neighbor items that are shown in bellowing figure. And some other methods may fail to model spatial and temporal relations of neighbor check-ins properly (see paper for more detail discussion).

![](img/aaai19/poi-intervals.PNG)

Method: 

![](img/aaai19/poi-lstm.PNG)

Shown in the above figure, based on LSTM, the authors also add two time interval gates and  two distance gates (no direction info) besides the input gate.

![](img/aaai19/poi-gates.PNG)

Gates $T1, D1$ are used to  filter new information considering influence of time interval and distance and then the filtered information is transferred to the hidden state and finally influences the next recommendation. Gates $T2, D2$ are used in a similar way for estimating new cell state.

