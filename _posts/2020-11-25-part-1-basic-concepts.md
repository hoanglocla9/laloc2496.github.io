---
layout: post
title: Cache Modeling ~ Part 1 - Basic concepts
tags: [modeling, queue system, random process]
comments: true
---


## 1. Introduction
The caching mechanism occurs in many computer sciences' fields, from low-level memories of physical machines to global-scale content delivery networks. Modeling caching systems helps us get insights into the systems' behaviors, and from these knowledges, we can tune and improve the performance.\
Mathematical models of caching systems have been considered since the early of 2000s. One of the most famous models is proposed by H.Che in [1]. In particular, H.Che assumed that arrival requests natively follow a stationary Poisson process. In specific, he assumes that the arrival event of specific content at a leaf cache will have the same Bernoulli distribution at any time, which is unrealistic, especially in the case of live-streaming service and the dynamic of contents through time. Originally, his model is designed for the LRU eviction algorithm and extended for several other eviction algorithms by V. Martina in [2]. Relaxed-constraints versions of this model are proposed in several works as [3], [4]. In this series's scope, I will only describe the ideas of H.Che's model.\
*Note: I assume that you have basic knowledge of probability theory. For the sake of brevity, the next section only recaps some random process concepts, which are useful to develop the model's ideas.*

## 2. Some concepts of random process
The model is built based on probability theory. So, firstly, I will recap some related probability definitions. To simplify the probability concepts without rigorous mathematical statements, we will consider the following example. To evaluate the current business situation of a bakery. The store's owner observes customers who arrive at his/her store. The time is discretized into equal intervals, such as hour, minute, or second. We denote the probability that a customer arrives at the store is p and the arrival event at time t is $$X_{t} \in {0, 1}$$. Assuming that an arrival occurs independently and with equal probability p at every time. Clearly, $$X \sim Bernoulli(p)$$. We say that an arrival event happens when $$X_{t} = 1$$.

**a. Random process:** \
A Bernoulli process is a sequence of independent trials, in which a trial is an event that a customer arrives at the store or not at the time t. Formally, A Bernoulli process is a sequence $$X_{1}, X_{2}, X_{3},...$$ of independent Bernoulli variables. More general, A random process is a collection of random variables usually indexed by time. The time can be discrete or continuous. \
In formal, the random process is defined as follows: 
Let $$S(t)$$ is a random variable. We collectively consider values of $$S(t)$$ at the time $$t \in (-\infty, +\infty)$$. $$\{S_{t}\}$$ is called a random process or a stochastic process. We can say that the random process $$\\{S_{t}\\}$$ is indexed by the set of time $$t \in J$$. ($$J$$ usually is a subset of the real line). \
*Note: In some documents, the authors denote $$t \in [0, +\infty)$$ to consider the process with $$t=0$$ is the current time.*

**b. Arrival Time** \
Now, after having the probability that a customer visits the store, the owner wants to know with a given time, how many times that an arrival event happens.\
Denote $$S_{t}$$ is the number of customers that have arrived at the store at time t. $$T_{k}$$ is the time that there are $k$ arrival events.\
$$S_{t} = \sum\limits_{i=0}^t X_{i}$$\
We can see that $$S_{t}$$ is a random variable with a binomial distribution by its definition. Clearly, $$T_{k}$$ is a random variable, and the above question can be answered if we know the distribution of variable $$T_{k}$$. We assume that the $$k^{th}$$ arrival event occurs at time $$n^{th}$$. This assumption is equivalent to that there are exactly $$k-1$$ arrival events before time n and at time $$n$$, an arrival event happens. Mathematically, we formulate the above statement as:\
$$\{T_{k} = n\} = \{X_n = 1, S_{n-1} = k-1\}$$ \ 
Because, $$X_{i}$$ is independent so that $$X_{n}$$ and $$S_{n-1}$$ are independent of each other. With $$n \geq k$$, we have: \
$$
\begin{align}
P(T_{k}=n) & = P(X_{n}=1)*P(S_{n-1}=k-1) \\
 & = p * [C_{n-1}^{k-1} * p^{k-1} * (1-p)^{n-k}] \\
& = C_{n-1}^{k-1} * p^{k} * (1-p)^{n-k}
\end{align}
$$ \
Clearly, $$T_{k}$$ are random variables that follow a negative binomial distribution.
> A typical difference between binomial and negative binomial distribution that the number of trials is fixed in the binomial distribution, meanwhile in the negative one, the number of arrivals is fixed.

Moreover, denoting $$Z$$ is the time that the first customer arrives at the store. With the similar analysis, we can derive that $$Z$$ is a random variable with Geometric distribution. In other words, $$Z \sim Geometric(p)$$.\
In the above analysis, we assume that time is discrete. In the case of continuous-time, the above analysis is still true. $$N_{t}$$ is denoted for the continuous-time version of $$S_{t}$$. Both $$\{N_{t}\}_{(t \geq 0)}$$ and $$\{S_{t}\}_{(t\geq0)}$$ are arrival processes. Moreover, $$\{N_{t}\}$$ is a Poisson process. In general, we have the arrival process's concept as follows:
* An arrival process is a random process with an indexed set of Bernoulli random variables.
* A Poisson process is a continous-time arrival process. Let λ is the intensity or arrival rate, $$N_{t} \sim Pois(λt)$$.

**c. Properties of Poisson process:** \
In this section, we will list some typical properties of Poisson process. Let random process $$\{N_{t}\}_{(t\geq0)}$$ is a Poisson process, we have:
* **Independence increments:** This property states that when we compute the number of arrivals from time $$s$$, we don't need to care about the number of arrivals that happen before that. In other words, the number of arrivals between time $$s$$ and $$t$$ only depends on the trials in this time interval and is therefore independent of the arrivals before time $$s$$. Formally, we have:\
$$
\begin{align}
N_{t} − N_{s} \perp\perp \{N_{r}\}_{(r \leq s)} \text{, for } t \geq s
\end{align}
$$
* **Stationary increments:** The second property a little bit seems like a time-shift invariant. In particular, it states that $$N_{t-s}$$ has the same distribution as $$N_{t} - N_{s}$$.\
$$
\begin{align}
N_{t} − N_{s} \sim Pois(λ(t − s)) \text{, for } t \geq s
\end{align}
$$ 

From the above properties, we can say a Poisson process is a strict-sense stationary random process.

**d. Inter-arrival time:** \
A question arises that "How long do we have to wait until the next customer arrives ?" or more specific, "What is the distribution of $$T_{k} - T_{k-1}$$? Can we compute exactly its value ?"\
Firstly, $$T_{k} - T_{k-1}$$ is a random variable, so that we can only compute its expected value. It is also called inter-arrival time. Moreover, the inter-arrival time of a Poisson process follows an exponential distribution. Formally, we have the following statements:
* Let $$\{N_{t}\}_{(t \geq 0)}$$ be an arrival process. The time of $$k^{th}$$ arrival of this process is called an arrival time. In other words, it is $$T_{k}$$.
* The time between two consecutive arrivals of $${N_{t}}_{t \geq 0}$$ is called an inter-arrival time. Again, in other words, it is $$T_{k} - T_{k-1}$$, for $$k \geq 1, T_{0} = 0.$$
* The inter-arrival time variables are independent and identically distributed (i.i.d). Especially, if $$N_{t} \sim Pois(λt)$$, $$T_{k}-T_{k-1} \sim Exp(λ)$$. 

In the general case, the random variable, which indicates the time between arbitrary $$k_{1}$$ and $$k_{2}$$, for $$k_{2} - k_{1}\geq 1$$, is distributed following a Gamma distribution. $$T_{k_{2}} - T_{k_{1}} \sim Gamma(k_{2}-k_{1}, λ)$$.

## 3. Conclusions:
In this part, I only recap some basic concepts and properties, which relate to Poisson process. In the next parts, I will introduce and describe the ideas of H.Che's analysis.

## 4. References:
[1] Che, Hao & Wang, Zhijung & Tung, ye. (2001). Analysis and design of hierarchical Web caching systems. Proceedings - IEEE INFOCOM. 3. 1416–1424 vol.3. 10.1109/INFCOM.2001.916637.\
[2] Garetto, Michele & Leonardi, Emilio & Martina, Valentina. (2016). A Unified Approach to the Performance Analysis of Caching Systems. ACM Transactions on Modeling and Performance Evaluation of Computing Systems. 1. 1–28. 10.1145/2896380. \
[3] Leonardi, Emilio & Torrisi, Giovanni. (2014). Least Recently Used caches under the Shot Noise Model. 10.1109/INFOCOM.2015.7218615. \
[4] Garetto, Michele & Leonardi, Emilio & Traverso, Stefano. (2015). Efficient analysis of caching strategies under dynamic content popularity. 2263–2271. 10.1109/INFOCOM.2015.7218613. \
[5] Probability Course, <https://www.probabilitycourse.com/chapter10/10_1_4_stationary_processes.php>, [Accessed-online: 21/11/2020]. \
[6] Ramon van Handel (2016), Probability and Random Processes, ORF 309/MAT 380 Lecture Notes Princeton University.
