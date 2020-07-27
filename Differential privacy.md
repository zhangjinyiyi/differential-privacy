# Differential privacy

[TOC]

## Key concepts

- Neighboring dataset $d$ and $d'$
- Sensitivity $\Delta f$ 
- Privacy loss
- Log of moment generation function $\alpha(\lambda)$ 
- $(\epsilon, \delta)$-differentially private mechanism: statistical interpretation of $\epsilon$ and $\delta$ 
- Composition theorem: ordinary, strong (different version available), moment accountant
- Privacy amplification via sampling
- Paradigm for deep learning with differential privacy: addiative noise to gradient update, PATE [Pâté]

## Questions

### Key questions to be answered

- For a given database with a centain distribution and operation/query/repeated queries, how to give desired privacy loss, i.e. $\epsilon$ and $\delta$ ?
  - Data-based privacy analysis in PATE is useful.
- Closed-loop privacy control design: given a desired cumulative privacy budget, how to design the repeated procedure and give the desired privacy budget for each step?
  - Moments accountant: Theorem 2.1 and 2.2 gives hints on this point.
- Given $\epsilon$ and $\delta$, how to determine which distribution to use? How to determine these distribution parameters such as $\sigma$, in order to make the $(\epsilon, \delta)$-differentially private ?
  - The selection of distribution remains an open question.
  - Different composition theorems give different bound to privacy loss estimate, as a result, different 
- Why adding an extra term $\delta$ to the original $\epsilon$-DP definition? It serves as a relaxation to the original definition, but for what ? It facilate the mechanism design?
  - E.g. for Gaussian distribution, it is impossible to make $\delta$. In the case of Theorem 2.2, $\delta$ literablly makes any choice of $\epsilon$ possible. A deep understanding would be helpful for later advanced usage.

### Others

- How to create a PointerTensor from local to remote in the remote node? then send it to local node for operation?
  - .search(tag, lable) 
- Comparision: Federated learning, deep learning with DP and FL, deep learning with DP and remote query/operation, PATE
  - ***Federated learning*** provides a certain level of protection to the original data, but no guarentee for any adversary attack with gradient information
  - ***Deep learning with DP and remote query***: only DP private gradient update is send back
    - Can be done with remote excution: like a query to a database
  - ***Deep learning with DP and FL***: with multiple nodes, aggregation of several different gradient updates
  - ***PATE***: local teacher model, central student model, use the DP teacher model result to train the student model
    - voting, but in a DP way
  - Secure federated learning
- How to interprete the crime survay as a DP problem with $(\ln3, 0)$ 
- What does the distance mean for a multi-row and multi-column database?
  - $$||x-y||_{1} \le 1$$

## Differential privacy and machine learning

> From Brendan McMahan
>
> In general, machine learning and privacy fit together nicely.
>
> ... especially differential privacy, is about taking some of that existing ideas around ***overfitting*** and then kind of formalizing and making sure that we can prove that we are not overfitting to any one example in the dataset.
>
> Privacy in machine learning, is to prevent the learned model from memorizing someone's data and that some other people could reconstruct your data from the model.

## Federated learning and model IP issues

- encryption
- homomorphous encryption
- secure enclave
- Trusted hardware

still an open problem

## Intuition to differential privacy

Differential privacy defines a **property** for ***privacy protection algorithms/mechnisms***, either with a trusted aggregator or without a trusted aggregator (where SMC is useful). For those that satisfy differential privacy, namely ***differentially private mechanism***, promise:

> You will not be affected, adversely or otherwise, by allowing your data to be used in any study or analysis, no matter what other studies, are available.

It means that any statistical analysis based on dataset done via ***differentially private mechanism*** does not compromise the privacy of any individual contained within that dataset. 

### Different solutions to achieve DP?

Popular privacy protection mechnisms may include:

- Secret sharing (SS): split data into shares
- Homomorphic encryption (HE): encrypting data
  - Three schemes: Paillier, Acs, and Shi
- Perturbation-based (PB): perturbing data with significant noise to achieve DP: Gaussian or Laplacian
  - ***[This is what most papers on DP is working on]***

A comparison of computation time for sum aggregation is given as follows:

<img src="/Users/yi/Library/Application Support/typora-user-images/image-20200306114809735.png" alt="image-20200306114809735 | width=100" style="zoom:60%;" />

The result shows that, when achieving similar differential privacy, HE takes 10 to 10000 times more computation time than PB, and SS takes ~100 times more computation time than PB. For devices with limited computation power or battery, e.g. edge computing device, mobile, cars, etc., PB methods are extremely useful.

## Local differential privacy

Noise is added to each individual record. This can be used when there is no trust between data provider (i.e. each individual) and data holder. Noise is added to each data record. A typical example is the flip-coin survey.

## Global differential privacy

There is trust between data privider and data holder, but no trust between data holder and data analysist (i.e. the potential data end user, who tried to conduct statistical analysis or training/learning on the dataset). The noise is added to the output of mechnisms.

## $\epsilon$-differential privacy

Given the property defined by differential privacy, $\epsilon$-differential privacy takes a step further, defines a **measure** $\epsilon$ of how much information is leaked by an algorithm or a mechanism $\mathcal{M}$. This permits comparisons among different techniques: for a fixed bound on privacy loss, which technique provides better accuracy? For a fixed accuracy, which technique provides better privacy?

The formal definition is

> A randomized algorithm $$\mathcal{M}$$ with domain $$\mathbb{N}^{|\mathcal{X}|}$$ is $$\epsilon$$-differential privacy if for all $$\mathcal{S}\subseteq Range({\mathcal{M}})$$ and for all $$x, y \in \mathbb{N}^{|\mathcal{X}|}$$ such that $$||x-y||_{1} \le 1$$:
> $$
> Pr[\mathcal{M}(x)\in \mathcal{S}] \le e^{\epsilon}\cdot Pr[\mathcal{M}(y)\in \mathcal{S}]
> $$

Note that $||x||_1$ is a measure of size of a database $x$ (i.e. the number of records it contains), and $||x-y||_1$ is a measure of how many records differ between $x$ and $y$. 

$\epsilon$ actually gives an upper bound of probability change for two neighboring database (a single individual's data is changed).

> To be confirmed/clarified:
>
> The definition gives a quantitative comparison between WITH/WITHOUT contribution from a certain people.
>
> - Left-hand side $Pr[\mathcal{M}(x)\in S]$ means the case where a certain record provides USEFUL information in the final analysis; 
>   - Probability of complete privacy leakage
> - Left-hand side $Pr[\mathcal{M}(y)\in S]$ means the case where a certain record provides NON-USEFUL information in the final analysis, which will finally be removed with the noise during statistical analysis;
>   - Probability of no privacy leakage: only noise is given for the final analysis; ***privacy is protected by random noise***.

The ideal case where no privacy is leaked, comes when $\epsilon=0$. In this case, the outputs of $\mathcal{M}$ for two adjacent database $x$ and $y$ have exactly the same probability distribution. This implies that no information could be infered by any kind of differential attack, which achieves perfect protection of individual privacy in the database.

> **Perfect privacy**
>
> A query to a database ruturn the same value even if we remove any person from the database.

When $\epsilon$ is stricly positive, it means a certain level of statistical information could be leaked by the given $\mathcal{M}$. The bigger $\epsilon$ is, the more information $\mathcal{M}$ leaks, the less effective is the mechanism. 

## $$(\epsilon,\delta)$$-differential privacy

$$(\epsilon,\delta)$$-differential privacy provides a relaxition form of $\epsilon$-differential privacy with an extra term $\delta$, defined as:

> A randomized algorithm $$\mathcal{M}$$ with domain $$\mathbb{N}^{|\mathcal{X}|}$$ is $$(\epsilon,\delta)$$-differential privacy if for all $$\mathcal{S}\subseteq Range({\mathcal{M}})$$ and for all $$x, y \in \mathbb{N}^{|\mathcal{X}|}$$ such that $$||x-y||_{1} \le 1$$:
> $$
> Pr[\mathcal{M}(x)\in \mathcal{S}] \le e^{\epsilon}\cdot Pr[\mathcal{M}(y)\in \mathcal{S}]+\delta
> $$

The only difference compared to $\epsilon$-differential privacy is that an extra term $\delta$ is added. The extra term implies that no matter how the mechanism is designed, there is always a certain amount of privacy leakage. , even when $\epsilon=0$. This breaks the plain $\epsilon$-differential privacy.

Typically, the value of $\delta$ is less than the inverse of any polynomial in the size of database.

## Privacy Loss

For neighboring databases $x, y \in \mathcal{D}^n$, a randomized mechanism $\mathcal{M}: \mathcal{D}^n\rightarrow \mathbb{R}$, auxiliary input $aux$, and an outcome $\xi \in \mathbb{R}$ , the privacy loss $l$ at mechanism output $\xi$ is defined as
$$
\mathcal{l}(\xi;\mathcal{M}, x, y, aux) = ln\bigg(\frac{Pr(\mathcal{M}(x)=\xi)}{Pr(\mathcal{M}(y)=\xi)}\bigg)
$$
Privacy loss $\mathcal{L}$ is a function of random variable $\Xi$, so $\mathcal{L}$ is also random variable. From the definition of $(\epsilon, \delta)$-differential privacy, it can be conclude from Lemma 3.17 [DP Bible] that
$$
Pr(|\mathcal{L}| \le \epsilon) = 1 - \delta
$$

## Important properties of $$(\epsilon,\delta)$$-differential privacy

### Composition theorem

The quantification of loss permits the analysis and control of cumulative privacy loss over multiple computations. Understanding the behavior of differentially private mechanisms under composition enables the design and analysis of complex differentially private algorithms from simpler differentially private building blocks.

### Group policy

Differential privacy permits the analysis and control of privacy loss incurred by groups, such as families.

### Closure under post-processing

Differential privacy is immune to post-processing: A data analyst, without additional knowledge about the private database, cannot compute a function of the output of a differentially private algorithm $\mathcal{M}$  and make it less differentially private. That is, a data analyst cannot increase privacy loss, either under the formal definition or even in any intuitive sense, simply by sitting in a corner and thinking about the output of the algorithm, no matter what auxiliary information is available.

## $(\epsilon, \delta)$-differentially private mechanism design

The randomized algorithm/mechanism $\mathcal{M}$ is generally a combination of original operation/query we want to execute on the original dataset and a certain amount of noise. 

How much noise are we going to add to the outputs? The answer is the minimum amount of noise requires to satisfy the **privacy budget/constraints** termed by $\epsilon$ and $\delta$, which depends on:

- Type of noise: Laplacian, Gaussian, etc.
- Sensitivity $\Delta f$ of original operation/query $f$
- Desired $\epsilon$
- Desired $\delta$ 

In general , when $\epsilon$ is small, more privacy should be preserved, more noise should be combined into the query outputs.

A common paradigm is to add noise $\mathcal{N}$ to the original query $f: \mathcal{D}\rightarrow \mathbb{R}$:
$$
\mathcal{M} = f(\mathcal{D})+\mathcal{N}
$$
The noise $\mathcal{N}$ is designed to follow a certain distribution, such as Laplacian and Gaussian distribution. In order to specify the distribution, it is necessary to determine the distribution parameters, e.g. mean $\mu$ and  standard deviation $\sigma$ for Gaussian distribution. Those parameters shall be expressed as a function of sensitivity $\Delta f$, $\epsilon$ and $\delta$ under some criteria which eventually make the randomized mechanism with given noise $(\epsilon, \delta)$-differentially private.  

The determination of those PDF paramteres is basically the main job to do during mechnism design. During this step, the most difficult part is to find relationship between parameters, sensitivity $\Delta f$, $\epsilon$ and $\delta$. 

## Sensitivity

- $l_1$ sensitivity
- $l_2$ sensitivity

let $d$ be a positive integer, $\mathcal{D}$ be a collection of datasets, and $f: \mathcal{D}\rightarrow \mathbb{R}^d$ be a query function (deterministic, real valued). The sensitivity of a function, denoted $\Delta f$, is defined by
$$
\Delta f = max||f(\mathcal{D_1})-f(\mathcal{D_2})||_1
$$
where the maximum is over all paris of datasets $\mathcal{D_1}$ and $\mathcal{D_2}$ in $\mathcal{D}$ differing in at most one element and $||\cdot||_1$ denotes the $l_1$ norm.

The sensitivity of a function gives an ***upper bound*** on how much we must perturb its output to preserve privacy. One noise distribution naturally lends itself to differential privacy.

## Gaussian mechanism

let $\epsilon \in (0, 1)$ be arbitrary. For $c^2 > 2log(1.25/\delta)$, the Gaussian Mechanism with parameter $\delta \ge c\Delta_2f/\epsilon$ is $(\epsilon, \delta)$-differentially private.

## Laplacian mechanism

### Laplacian distribution

The PDF of laplacian distribution with location parameter $\mu$ and scale parameter $b$:
$$
Lap(x|\mu, b)=\frac{1}{2b}exp\bigg(-\frac{|x-\mu|}{b}\bigg)
$$
Whose mean is $\mu$ and variance is $2b^2$.

![Probability density plots of Laplace distributions](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0a/Laplace_pdf_mod.svg/325px-Laplace_pdf_mod.svg.png)

### Laplacian mechanism

Given desired privacy constraints $\epsilon$ and $\delta$, sensitivity of original operation/query $\Delta f$, the scale parameter of Laplacian noise is calculated as follows to satisfy the $(\epsilon, 0)$-privacy constraints: [TODO: How? Proof]
$$
b=\frac{\Delta f}{\epsilon}
$$

> $\delta$ is always set to zero for Laplacian mechnism.

Mechanism with Laplacian noise is called Laplacian mechanism.

### Proof

Let $x \in \mathbb{N}^{|\mathcal{X}|}$ and $y \in \mathbb{N}^{|\mathcal{X}|}$ be such that $||x-y||_1 \le 1$, and let $f(\cdot)$ be some function $f:\mathbb{N}^{|\mathcal{X}|} \rightarrow \mathbb{R}^k$. Let $p_x$ denote the probability density function of $\mathcal{M}_L(x, f, \epsilon)$, and let $p_y$ denote the probability density function of $\mathcal{M}_L(y, f, \epsilon)$. We compare the two at some arbitrary points $z \in \mathbb{R}^k$.
$$
\begin{equation}
\begin{split}
\frac{p_x(z)}{p_y{(z)}} & = \prod_{i=1}^{k} \bigg(\frac{exp(-\frac{\epsilon|f(x)_i-z_i|}{\Delta f})}{exp(-\frac{\epsilon|f(y)_i-z_i|}{\Delta f})}\bigg) \\
& = \prod_{i=1}^{k} {exp\bigg(\frac{\epsilon\cdot(|f(y)_i-z_i|-|f(x)_i-z_i|)}{\Delta f}\bigg)} \\
& \rightarrow \ {triangle\ inequality} \\
& \le \prod_{i=1}^{k} {exp\bigg(\frac{\epsilon\cdot|f(y)_i-f(x)_i|}{\Delta f}\bigg)} \\
& = exp\bigg(\frac{\epsilon\cdot||f(x)-f(y)||_1}{\Delta f}\bigg) \\
& \rightarrow \ {definition\ of\ l_1 \ sensitivity definition} \\
& \le exp(\epsilon)
\end{split}
\end{equation}
$$

## Other mechanisms

## Composition theorem: privacy accounting method

We use composition theorem when we need repeated use of differentially private mechanisms on the same database or different databases. It helps us to estimate the cumulative privacy loss during repeated use of DP. 

From ***composition theorem*** to ***advanced/strong*** to ***moment accountant***, given a desired cumulative privacy budget for the entire analysis or training lifetime, the privacy loss limit for each mechanism becomes looser and looser, i.e. bigger $\epsilon$ for the mechanism of each step, then the additive noise could be reduced for each step, e.g. smaller variance $\sigma$. As a result, it is possible to achieve higher accuracy during statistical analysis or training.

### Composition theorem - The loosest one

Let $\epsilon, \delta \ge 0$, the class of $(\epsilon, \delta)$-differentially private mechanisms under $k$-fold adaptive composition provides $(k\epsilon, k\delta)$-differentially privacy.

### Privacy amplification via sampling

**Lemma 4 from [Beimel, 2010]**: Over a domain of datasets $\mathcal{T}^n$, if an algorithm $\mathcal{M}$ is $\epsilon\le 1$ differentially private, then for any dataset  $\mathcal{D} \in \mathcal{T}^n$, excuting $\mathcal{A}$ on uniformly random $q n$ entries (sampling rate is $q$) of $\mathcal{D}$ ensures $2\gamma\epsilon$-differential privacy. 

In case of $(\epsilon, \delta)$-DP, after sampling with $q$, $\rightarrow (2q\epsilon, q\delta)$-DP.

### Advanced/strong composition theorem

Let $\epsilon, \delta \ge 0$, the class of $(\epsilon, \delta)$-differentially private mechanisms under $k$-fold adaptive composition provides $(\epsilon', k\delta+\delta')$-differentially privacy for 
$$
\epsilon'= \sqrt{2k\cdot ln(1/\delta')}\epsilon + k\epsilon(e^{\epsilon}-1)
$$

### Improved advanced composition theorem

Let $\epsilon, \delta \ge 0$, the class of $(\epsilon, \delta)$-differentially private mechanisms under $k$-fold adaptive composition provides $(\epsilon', k\delta+\delta')$-differentially privacy for 
$$
\epsilon'= \sqrt{2k\cdot ln(1/\delta')}\epsilon + k\epsilon(e^{\epsilon}-1)/2
$$

### Moment accountant for SGD 

Details could be found below.

## Moment accountant for SGD: details

### Useful concept

- Privacy loss definition
- $(\epsilon, \delta)$ -DP definition
- Privacy amplification via sampling
- Advanced composition theorem
- Moments accountant: Algorithm 1, Theorem 2.1, Theorem 2.2, Lemma 3

### Questions

- Scaling effect: what if not Gaussian distribution?
- $\alpha(\lambda)$ with Gaussian noise: why E1>E2, like always?

### DP SGD Algorithm: 

<img src="/Users/yi/Library/Application Support/typora-user-images/image-20200311101117612.png" alt="image-20200311101117612 width" style="zoom:50%;" />

### Difference between new approach and paper approach in DL with DP

Given batch size of $m$, the paper approach gives
$$
\mathcal{M}(d)=\frac{1}{m}\bigg(\sum_{i=1}^{m}f(d_i) +X\sim\mathcal{N}(0,C^2\sigma^2)\bigg)
$$
If the noise is added to each individual gradient calculated, then the randomdized output becomes
$$
\begin{equation}\begin{split}
\mathcal{M}(d)
&=\frac{1}{m}\sum_{i=1}^{m}\bigg(f(d_i) +X\sim\mathcal{N}(0,C^2\sigma'^2)\bigg) \\
&=\frac{1}{m}\bigg(\sum_{i=1}^{m}f(d_i) + \sum_{i=1}^{m}X\sim\mathcal{N}(0,C^2\sigma'^2)\bigg) \\
&=\frac{1}{m}\bigg(\sum_{i=1}^{m}f(d_i) + m \cdot X \sim\mathcal{N}(0,C^2\sigma'^2)\bigg) \\
&=\frac{1}{m}\bigg(\sum_{i=1}^{m}f(d_i) + X'\sim\mathcal{N}(0,m^2C^2\sigma'^2)\bigg) \\
\end{split}\end{equation}
$$
Those two process are equivalent when
$$
\sigma=m\cdot\sigma'
$$
As a result, for a single query with Gaussian noise, given $\delta$, the resulting $\epsilon'$ is m times the original $\epsilon$, i.e. $\epsilon/m$.

For a composed mechanism like SGD, according to lemma 3 and Theorem 2.1 from [deep learning with DP]
$$
\alpha(\lambda) \le \frac{Tq^2\lambda^2}{\sigma^2}
$$
It is clear that the upper bound of DP-SGD becomes $1/m^2$ times of the original paper approach. 

### Why $\mu=q\mu_1+(1-q)\mu_0$ in Lemma 3?

Suppose $d$ and $d'$ are two neighboring database, $d=d \cup \{d_n\}$. Without loss of generality, $||f(d_n)||=1$, $\sum_{i\in J \backslash\{n\}}=0$. In a given provess, fix $d'$, $\mathcal{M}(d') \sim \mu_0$. 
$$
\begin{equation}\begin{split}\mathcal{M}(d) &= \sum_{i\in J} f(d_i) + \mathcal{N}\sim\mu_0 \\&= \bigg[\sum_{i\in J\backslash \{n\}} f(d_i)\bigg] + f(d_n)+ \mathcal{N}\sim\mu_0 \\&= \mathcal{M}(d') + f(d_n)\end{split}\end{equation}
$$
Since $d_n$ is sampled with a probability of $q$, $Pr(f(d_n)=1)=q$, as a result
$$
\begin{equation}\begin{split}Pr(\mathcal{M}(d)=z)&=Pr((\mathcal{M}(d')+f(d_n)=z) \cap \{f(d_n)=0, f(d_n)=1\}) \\&=Pr((\mathcal{M}(d')=z) \cap f(d_n)=0) + Pr((\mathcal{M}(d')+1=z) \cap f(d_n)=1)\\&=(1-q)\mu_0+q\mu_1\end{split}\end{equation}
$$

### Log of Moment Generation Function

Defined as $\alpha(\lambda)=log\ max(E_1, E_2)$
$$
\begin{equation} \begin{split}E_1  &=E_{z\sim\mu\ } \bigg[ \bigg( \cfrac{\mu(z)}{\mu_0(z)} \bigg)^{\lambda} \bigg] \\E_2  &=E_{z\sim\mu_0} \bigg[ \bigg( \cfrac{\mu_0(z)}{\mu(z)} \bigg)^{\lambda} \bigg]\end{split}\end{equation}
$$

### Shifting effect

By a replacement of variables, it can be proved that any shift $s$ for $\mu_0\sim \mathcal{N}(s,\sigma^2)$ and $\mu_1\sim \mathcal{N}(s+1,\sigma^2)$ has no influence on the value of $E_1$ and $E_2$. This effect could also be generalized to other distributions.

A simplified proof with $E_1$ is given here as an example: 

let $X, Y, X', Y'$ be four random variables, $s$ is the shift distance between $X$ and $X'$, $Y$ and $Y'$ such that $X'=X+s$, $Y'=Y+s$. $\mu_X, \mu_{X'}, \mu_Y$ and $\mu_{Y'}$ are the pdfs of $X, X', Y$ and $Y'$ respectively. Therefore, $\forall t\in \mathbb{R}$, $\mu_X(t)=\mu_{X'}(t+s)$, $\mu_Y(t)=\mu_{Y'}(t+s)$. $\mu$ is linear combination of $\mu_X$ and $\mu_Y$, $\mu'$ is linear combination of $\mu_{X'}$ and $\mu_{Y'}$.
$$
\begin{equation} \begin{split} E_1' &= E_{Z \sim\mu'} \bigg[ \bigg( \cfrac{\mu'(z)}{\mu_{X'}(z)} \bigg)^{\lambda} \bigg]\\&= \int_{-\infin}^{+\infin} \mu'(z) \bigg( \cfrac{\mu'(z)}{\mu_{X'}(z)} \bigg)^{\lambda} dz \\\\& \rightarrow \mu_X(t)=\mu_{X'}(t+s) \\& \rightarrow \mu_Y(t)=\mu_{Y'}(t+s) \\\\&= \int_{-\infin}^{+\infin} \mu(z-s) \bigg( \cfrac{\mu(z-s)}{\mu_X(z-s)} \bigg)^{\lambda} dz \\\\& \rightarrow x=z-s\\\\&= \int_{-\infin}^{+\infin} \mu(x) \bigg( \cfrac{\mu(x)}{\mu_X(x)} \bigg)^{\lambda} d(x+s)\\\\& \rightarrow \lim_{Q\to\infty} \int_{Q}^{+\infin} f(x) dx = 0\\\\&= \int_{-\infin}^{+\infin} \mu(x) \bigg( \cfrac{\mu(x)}{\mu_X(x)} \bigg)^{\lambda} d(x)\\&= \int_{-\infin}^{+\infin} \mu(z) \bigg( \cfrac{\mu(z)}{\mu_X(z)} \bigg)^{\lambda} d(z)\\&= E_{Z \sim\mu} \bigg[ \bigg( \cfrac{\mu(z)}{\mu_{X}(z)} \bigg)^{\lambda} \bigg]\\\end{split}\end{equation}
$$

### More shift, higher log moment

I partially verified it numerically in MATLAB with Gaussian distribution.

Proof could be given later.

### Scaling effect

Suppose a factor $C$ is used to scale up the $\mathcal{M}$, similar to the effect of clipping, then is we look at the Lemma 3, the resulting $\mu_0$ becomes the pdf of $\mathcal{N}(0,C^2\sigma^2)$ and $\mu_1$ becomes the pdf of $\mathcal{N}(C,C^2\sigma^2)$. If you follow the calculation of log moment in lemma 3, you will find that the value does not change either. 

This also applies to other continuous distribution.

I give a simple proof here.

An easy fact about pdf will be useful in the following proof: for a random variable $X$ and its pdf $\mu(x)$, 
$$
\int \mu(x) dx = 1
$$
If $X'=cX$ and its pdf is $\mu'(x)$, then we have to make sure that 
$$
\int \mu'(x) dx = 1
$$
Suppose $\mu'(x)=a\mu(x/c)$,
$$
\begin{equation} \begin{split}\int \mu'(x) dx = \int a\mu(x/c) dx = \int a\mu(t) d(c\cdot t) = c \int a\mu(t) dt = 1\end{split} \end{equation}
$$
As a result, $a=1/c$, then
$$
\mu'(x)=\frac{\mu(x/c)}{c}
$$
let $X, Y, X', Y'$ be four random variables, $c$ is the scaling between $X$ and $X'$, $Y$ and $Y'$ such that $X'=cX$, $Y'=cY$. $\mu_X, \mu_{X'}, \mu_Y$ and $\mu_{Y'}$ are the pdfs of $X, X', Y$ and $Y'$ respectively. Therefore, $\forall t\in \mathbb{R}$, $\mu_X(t)=c\cdot\mu_{X'}(c \cdot t)$, $\mu_Y(t)=c\cdot\mu_{Y'}(c\cdot t)$. $\mu$ is linear combination of $\mu_X$ and $\mu_Y$, $\mu'$ is linear combination of $\mu_{X'}$ and $\mu_{Y'}$.
$$
\begin{equation} \begin{split} E_1' &= E_{Z \sim\mu'} \bigg[ \bigg( \cfrac{\mu'(z)}{\mu_{X'}(z)} \bigg)^{\lambda} \bigg]\\&= \int_{-\infin}^{+\infin} \mu'(z) \bigg( \cfrac{\mu'(z)}{\mu_{X'}(z)} \bigg)^{\lambda} dz \\\\& \rightarrow \mu_X(t)=c\cdot\mu_{X'}(c \cdot t) \\& \rightarrow \mu_Y(t)=c\cdot\mu_{Y'}(c\cdot t) \\\\&= \int_{-\infin}^{+\infin} \frac{\mu(z/c)}{c} \bigg( \cfrac{\mu(z/c)/c}{\mu_X(z/c)/c} \bigg)^{\lambda} dz \\\\& \rightarrow x=z/c\\\\&= \int_{-\infin}^{+\infin} \frac{\mu(x)}{c} \bigg( \cfrac{\mu(x)}{\mu_X(x)} \bigg)^{\lambda} d(cx)\\&= \int_{-\infin}^{+\infin} \mu(x) \bigg( \cfrac{\mu(x)}{\mu_X(x)} \bigg)^{\lambda} d(x)\\&= \int_{-\infin}^{+\infin} \mu(z) \bigg( \cfrac{\mu(z)}{\mu_X(z)} \bigg)^{\lambda} d(z)\\&= E_{Z \sim\mu} \bigg[ \bigg( \cfrac{\mu(z)}{\mu_{X}(z)} \bigg)^{\lambda} \bigg]\\\end{split}\end{equation}
$$

### How to use shifting and scaling for analysis?

For example, let $\alpha(\mu_{a,c}, \mu_{a+b, c})$ be the max log moment of case with $\mu_{a,c}\sim\mathcal{N(a,c^2\sigma^2)}$ and $\mu_{a+b,c}~\mathcal{N(a+b,c^2\sigma^2)}$ as the basis for calculation, $b\le c$, then
$$
\begin{equation} \begin{split} \alpha(\mu_{a,c}, \mu_{a+b, c}) & \\&\overset{\mathrm{shift\ by\ -a}}{=} &\ \alpha(\mu_{0,c}, \mu_{b, c}) \\&\overset{\mathrm{scale\ by\ 1/c}}{=} &\ \alpha(\mu_{0,1}, \mu_{b/c, 1}) \\& \overset{\mathrm{shift:\ b/c \le 1}}{\le} &\ \alpha(\mu_{0,1}, \mu_{1, 1})\end{split} \end{equation}
$$

### Rewrite Lemma 3

Suppose that $f: \mathcal{D}\rightarrow\mathbb{R}^p$ with $||f(\cdot)||_2 \le C$, Let $\sigma \ge 1$ and let $J$ be a sample from $[n]$ where each $i \in [n]$ is choosen independently with probability $q<1/(16\sigma)$. Then for any positive integer $\lambda \le \sigma^2 ln(1/(q\sigma))$, the mechanism $M(d)=\sum_{i\in J} C\cdot f(d_i) + \mathcal{N}(0,C^2\sigma^2\bold{I})$ satisfies
$$
\alpha_{\mathcal{M}}(\lambda) \le \cfrac{q^2\lambda(\lambda+1)}{(1-q)\sigma^2+O(q^3\lambda^3/\sigma^3)}
$$

### $E_1$ formulation in TensorFlow code

$$
\begin{equation} \begin{split}E_1 &= E_{z\sim\mu\ } \bigg[ \bigg( \cfrac{\mu(z)}{\mu_0(z)} \bigg)^{\lambda} \bigg]\\		&= \int \mu(z) \bigg(\frac{\mu(z)}{\mu_0(z)}\bigg)^\lambda dz \\		\\		& \rightarrow \mu = q\mu_1+(1-q)\mu_0 \\		\\		&= q\int \mu_1(z) \bigg(\frac{\mu(z)}{\mu_0(z)}\bigg)^\lambda dz + (1-q)\int \mu_0(z) \bigg(\frac{\mu(z)}{\mu_0(z)}\bigg)^\lambda dz \\		&= q E_{z\sim\mu_1} \bigg[\bigg(\frac{\mu(z)}{\mu_0(z)}\bigg)^\lambda\bigg] + (1-q) E_{z\sim\mu_0} \bigg[\bigg(\frac{\mu(z)}{\mu_0(z)}\bigg)^\lambda\bigg] \\		\\		& \rightarrow shift\ the\ 1^{st}\ term\ from\ \mu_1\ to\ \mu_0,\ from\ \mu_0\ to\ \mu_{-1}, \\		& as\ a\ result\ ,\mu^{term\ 1}=q\mu_{0}+(1-q)\mu_{-1} \\		\\		&= q E_{z\sim\mu_0} \bigg[\bigg(\frac{q\mu_0(z)+(1-q)\mu_{-1}(z)}{\mu_{-1}(z)}\bigg)^\lambda\bigg] + (1-q) E_{z\sim\mu_0} \bigg[\bigg(\frac{q\mu_1(z)+(1-q)\mu_{0}(z)}{\mu_0(z)}\bigg)^\lambda\bigg]\end{split}\end{equation}
$$

### Why $E_1>E_2$?

No proof is given in the code. It is tested numerically by the author.

### Moment calculation implementation

Details in code and Lemma 3.

### High-dimension problem

Even through the gradient we are dealing with is a vector, it is easy to generalize the one-dimensional analysis to high-dimensional problem, bacause the noise is added **independently** for different dimension. The hypothesis in Lemma 3 is quite ideal, e.g. zero mean, unit vector, etc. When we deal with the real problem, in our case, SGD, we do not always have such ideal hypothesis underlined. Therefore, it is important to keep the shifting and scaling effect in mind to analyze cases with different cases. 

In the implementation, when using $\mu_0$ and $\mu_1$ for the calculation of log moment for each step, it is actually using the another max/upper bound as an estimation of log moment for one step of SGD.

### Definition and Theorems

#### Theorem 2.1: Moments accountant - Compatibility

Suppose that a mechanism $\mathcal{M}$ consists of a sequence of adaptive mechanism $\mathcal{M_1}, ... , \mathcal{M_k}$ where $\mathcal{M_i}:\prod_{j=1}^{i-1} \mathcal{R}_j \times \mathcal{D} \rightarrow \mathcal{R_i}$. Then, for any $\lambda$ 
$$
\alpha_{\mathcal{M}}(\lambda) \le \sum_{i=1}^{k} \alpha_{\mathcal{M_i}}(\lambda)
$$

#### Theorem 2.2: Moments accountant - Tail bound

For any $\epsilon > 0$, the mechanism $\mathcal{M}$ is $(\epsilon, \delta)$-differentially private for 
$$
\delta = min_{\lambda} exp(\alpha_{\mathcal{M}}(\lambda)-\lambda\epsilon)
$$

#### Theorem 1: $(\epsilon, \delta)$-differentially private mechanism for SGD

> This theorem is only used to prove that moment accountant could achieve a tighter bound than strong composition theorem, not really useful for implementation.

There exist constants $c_1$ and $c_2$ so that given the sampling probability $q=L/N$ and the number of steps $T$, for any $\epsilon < c_1q^2T$, SGD-algorithm is $(\epsilon, \delta)$-differentially private for any $\delta > 0$ if we choose
$$
\sigma \ge c_2 \frac{q\sqrt{Tlog(1/\delta)}}{\epsilon}
$$

### How to use moments accountant in SGD?

> It is useful to have a closed-loop version of privacy control.
>
> Top-down approach is much more natural.

#### 1. A top-down approach

Given target $\epsilon$ and $\delta$ for the entire procedure, e.g. SGD, given a moment $\lambda$, the reference $\alpha_{r}(\lambda)$ can be calculated by ***Theorem 2.2: Tail bound theorem*** 
$$
\alpha_r(\lambda) = ln(\delta) + \lambda\epsilon
$$
Suppose that a mechanism $\mathcal{M}$ consists of a sequence of adaptive mechanisms $\mathcal{M_{i, i\in[1, k]}}$, it is possible to design a series of $\alpha_{\mathcal{M_i}}(\lambda)$ such that 
$$
\sum_i \alpha_{\mathcal{M}_i}(\lambda)=\alpha_r{(\lambda)}
$$
According to ***Theorem 2.1: compositibility***, the final achieved $\alpha_{\mathcal{M}}(\lambda)$ by this sequence of mechanisms is always smaller than the reference $\alpha_{r}(\lambda)$. As a result, the final achieved mechanism will be a little bit tighter thant the target one. [Could be analyzed]

Now, with the designed series of $\alpha_{\mathcal{M_i}}$, it is possible to find a suitable pair of $(\epsilon_i, \delta_i)$ for each step, then to find a suitable $\sigma$ for the noise. Privacy amplification via sampling should be used in case of sampling.

#### 2. A bottom-up approach

Given target $\epsilon_i$ and $\delta_i$ for each step $i$, depending on the type of noise, we can calculate the correponding $\sigma_i$, then add noise into the clipped gradients $g_i \in [-C,C]$. Pay attention to the fact that the function to calculate $g_i$ is deterministic, once the database used for calculation is sampled. In case a Gaussian noise is used, the output $\xi_i$ of mechanism $\mathcal{M_i}$ follows a normal distribution  $\mathcal{N}(g_i, \sigma^2)$. 

## Differential privacy for deep learning

> Perfect privacy for AI model
>
> Training a model on a dataset should return the same model even if we remove any person from the training dataset.

A main contribution from this paper is moment accountant, in my opinion, a tighter composition theorem compared to strong composition theorem when given the privacy loss for each step.

### Challenges

- Do we already know where "people" are reference in the dataset?
- Neural models rarely even train to the same location, even when trained on the dataset twice.

## PATE: Private Aggragation of Teacher Ensembles

### Questions

- Different queries: different epsilon, not realized yet
- Semi-superviser learning can achieve higher accuracy than aggregation accuracy?



- How to implement PATE?

  - Follow the procedure: quiet natural

- How to use data-dependant privacy analysis?

- How to implement data-dependant privacy analysis

  

- Page 6, Theorem 2: why the laplacian mechanism is $(2\lambda, 0)$-DP with noise $Lap(1/\lambda)$? Maybe it comes from a sensitivity of 2, but why 2? 

  - The laplacian noise is added to the voting count. If one sample changes the output of one teacher model output from one class to another, then two coordinate of count will be changed, as a result, sensitivity will be 2.

- Page 6, after theorem 2, Strong composition theorem: supposing a small $\lambda$, take only the 1st and 2nd order terms. Is it a suitable treatment?

  - Tayler series

- Understand the meaning of sampling rate q: for each sample, it has a probability of q to be sampled and (1-q) to be excluded in the sample. useful for the analysis of M(d) and M(d') in Lemma 3 for moment calculation

- def logmgf_exact_torch(q, priv_eps, l): what is the second term?

- GAN: find some paper to understand its logics

### Overview

![image-20200305164440783](/Users/yi/Library/Application Support/typora-user-images/image-20200305164440783.png)

PATE proposes a new high-level machine learning paradigm with differential privacy. It is different from federated learning and deep learning with differential privacy proposed in Abadi's paper. Some key highlights:  

- Post privacy analysis is implemented in PySyft: [Link to Github code](https://github.com/OpenMined/PySyft/tree/master/syft/frameworks/torch/dp)
- Hypothesis: the approach combines, in a black-box fashion, multiple models trained with ***disjoint*** datasets, such as records from different subsets of users.
- Key words: GAN, DP-Laplacian noise, **data-based private analysis**, moments accountant, strong composition theorem.

### Gap

***Gap*** measures the difference in votes between the most popular label and the second most popular label. The gap itself increases with the number of teachers, having more teachers would lower the privacy cost.

### Which hyperparameter matters in PATE?

- Number of teachers
  - If small, small gap, accuracy will be easily affected by noise
  - If big, small training set for each teacher model, then lower accuracy for each teacher model
- $\gamma$ for Laplacian noise
- Number of queries: as few as possible, this is why semi-supervised training approach is used
  - Query: number of times to qurey the teach model
  - [TODO: DISTILLATION, active learning]

### Theorem 2 and 3 [TODO]

### Data-dependant Privacy analysis

Privacy log moments are themselves raw data dependent (should be calculated with count matrix without noise), the final ε is therefore itself data-dependent and should not be revealed. This lead to the sensitivity smoothing

```
def perform_analysis(teacher_preds, indices, noise_eps, delta=1e-5, moments=8, beta=0.09)
```

#### How can privacy analysis used in real case?

Is this can be used in a closed loop manner?

![image-20200314113400009](/Users/yi/Library/Application Support/typora-user-images/image-20200314113400009.png)

#### Privacy analysis

Privacy mainly returns data-independant $\epsilon$ and data dependant $\epsilon$ 
- Data independant: the given $\epsilon$
- Data dependant: 
  - Data dependant log moment upper bound: minimum of three different bounds
  - Epsilon is calculated from by Theorem 2.2 in [DL with DP]
  - Sensitivity smoothing: log moment sensitivity, beta-smooth [Smooth Sensitivity and Sampling in Private Data Analysis]

#### Log moment upper bound

> *logmgf_exact: calculate minimum upper bound, not the real log moment*

Supposing Laplacian noise $Lap(1/\gamma)$, in PATE, $(\epsilon=2\gamma)$-DP. Let $q$ be the probability of non-optimal output.

**1. Theorem 2**
$$
\alpha(\lambda) \le 2\gamma^2\lambda(\lambda+1)
$$
**2. Original composition theorem**
$$
\alpha(\lambda) \le 2\gamma\lambda
$$
**3. Theorem 3**: Bound from Paper [Concentrated Differential Privacy: Simplifications, Extensions, and Lower Bounds] [TODO: understanding its meaning].

If $q<0.5$
$$
\alpha(\lambda) \le (1-q)\bigg(\frac{1-q}{1-exp(2\gamma)q}\bigg)^\lambda
$$
if $q \ge 0.5$
$$
\alpha(\lambda) \le 2\gamma \lambda
$$
##### Data-independent log moment function

$q$ is set to $1$, as a result, $bound_1$ and $bound_2$ is used.

```python
def logmgf_exact(q, priv_eps, l):
    """Computes the logmgf value given q and privacy eps.

  The bound used is the min of three terms. The first term is from
  https://arxiv.org/pdf/1605.02065.pdf.
  The second term is based on the fact that when event has probability (1-q) for
  q close to zero, q can only change by exp(eps), which corresponds to a
  much smaller multiplicative change in (1-q)
  The third term comes directly from the privacy guarantee.
  Args:
    q: pr of non-optimal outcome
    priv_eps: eps parameter for DP
    l: moment to compute.
  Returns:
    Upper bound on logmgf
  """
    if q < 0.5:
        t_one = (1 - q) * math.pow((1 - q) / (1 - math.exp(priv_eps) * q), l)
        t_two = q * math.exp(priv_eps * l)
        t = t_one + t_two
        try:
            log_t = math.log(t)
        except ValueError:
            print("Got ValueError in math.log for values :" + str((q, priv_eps, l, t)))
            log_t = priv_eps * l
    else:
        log_t = priv_eps * l

    return min(0.5 * priv_eps * priv_eps * l * (l + 1), log_t, priv_eps * l)
```

#### Non-optimal output probability $q$ : Lemma 4

Non-optimal output probability is the probability that the output is not the original winner.
$$
q=Pr(output \ne true\ winner)
$$
Let $n$ be the counts vector with $m$ elements [TODO: raw], and the addiative noise is $(\epsilon)-DP$ [TODO: can be extended to $(\epsilon, \delta)$]. The winner label $j^{\ *}$ is $j^{\ *} = arg\ max_i\ n_i$, then the Non-optimal output probability is defined as
$$
q \le \sum_{i\in [m]\backslash\{j\}} \frac{\epsilon(n_{j^{*}}-n_i)/2+2}{4\cdot exp(\epsilon(n_{j^{*}}-n_i)/2)}
$$
[TODO] http://mathoverflow.net/questions/66763/tight-bounds-on-probability-of-sum-of-laplace-random-variables

#### Smoothed sensitivity

More details in Corollary 2.4 [Page 7, Nissim, 2007]

Since the data dependant log moment/epsilon contains information from data, so a smoothing procedure is proposed to preserve privacy.

***Corollary 2.4***: (Calibrating Noise to smooth bounds on the sensitivity, 1-dimensional case). Let $f: \mathcal{D}^n \rightarrow \mathbb{R}$ be any real-valued function and let $S: \mathcal{D}^n \rightarrow \mathbb{R}$ be a $\beta$-smooth upper bound on the local sensitivity of $f$. Then

- If $\beta\le \frac{\epsilon}{2(\gamma+1)}$ and $\gamma > 1$, the algorithm $x \rightarrow f(x)+2(\gamma+1)S(x)/\epsilon$, where $\eta$ is sampled from the distribution with density $h(z) \sim 1/(1+|z|^\gamma)$, is $\epsilon$-differentially private.

- If $\beta\le \frac{\epsilon}{2ln(2/\delta)}$ and $\delta \in (0,1)$, the algorithm $x \rightarrow f(x)+\frac{2S(x)}{\epsilon}\eta$, where $\eta \sim Lap(1)$, is $(\epsilon, \delta)$-differentially private.

 In the code, the parameters needed for this are $\beta, \delta, \epsilon$.

```python
for i in indices:

        total_log_mgf_nm += torch.tensor(
            [logmgf_from_counts_torch(counts_mat[i].clone(), noise_eps, l) for l in l_list]
        )

        total_ss_nm += torch.tensor(
            [smooth_sens_torch(counts_mat[i].clone(), noise_eps, l, beta) for l in l_list],
            dtype=torch.float,
        )
```

```python
def smoothed_sens(counts, noise_eps, l, beta):
    """Compute beta-smooth sensitivity.

  Args:
    counts: array of scors
    noise_eps: noise parameter
    l: moment of interest
    beta: smoothness parameter
  Returns:
    smooth_sensitivity: a beta smooth upper bound
  """
    k = 0
    smoothed_sensitivity = sens_at_k(counts, noise_eps, l, k)
    while k < max(counts):
        k += 1
        sensitivity_at_k = sens_at_k(counts, noise_eps, l, k)
        smoothed_sensitivity = max(smoothed_sensitivity, math.exp(-beta * k) * sensitivity_at_k)
        if sensitivity_at_k == 0.0:
            break
    return smoothed_sensitivity
```

```python
def sens_at_k(counts, noise_eps, l, k):
    """Return sensitivity at distane k.

    Args:
      counts: an array of scores
      noise_eps: noise parameter used
      l: moment whose sensitivity is being computed
      k: distance
    Returns:
      sensitivity: at distance k
    """
    counts_sorted = sorted(counts, reverse=True)
    if 0.5 * noise_eps * l > 1:
        print("l too large to compute sensitivity")
        return 0
    # Now we can assume that at k, gap remains positive
    # or we have reached the point where logmgf_exact is
    # determined by the first term and ind of q.
    if counts[0] < counts[1] + k:
        return 0
    counts_sorted[0] -= k
    counts_sorted[1] += k
    val = logmgf_from_counts(counts_sorted, noise_eps, l)
    counts_sorted[0] -= 1
    counts_sorted[1] += 1
    val_changed = logmgf_from_counts(counts_sorted, noise_eps, l)
    return val_changed - val
```

### Result

- ***Improved deep learning with DP***: Our MNIST and SVHN students with (ε, δ) differential privacy of (2.04, 10−5) and (8.19, 10−6 ) achieve accuracies of 98.00% and 90.66%.

## DP cases

### Apple face ID, Typo recommendation, emoji recommendation

- All the face data stay on the devices, not on iCloud.

### Crime survey

A local $(\ln3, 0)$-DP case

## More to discover

- Private preserving machine learning
- Secure multi-party computation
- Federated transfer learning
- Secure federated learning
- Distributed learning
- Cryptography
- Homomorphic encryption

## Code

Tensorflow: https://github.com/mldbai/tensorflow-models/tree/master/differential_privacy

## Framework

- PySyft
- [TODO: another framework with a lot functionalities] FATE

## References

Abadi, M., Chu, A., Goodfellow, I., McMahan, H. B., Mironov, I., Talwar, K., & Zhang, L. (2016, October). Deep learning with differential privacy. In *Proceedings of the 2016 ACM SIGSAC Conference on Computer and Communications Security* (pp. 308-318).

- SGD with DP
- Moments accountant

Aono, Y., Hayashi, T., Wang, L., & Moriai, S. (2017). Privacy-preserving deep learning via additively homomorphic encryption. *IEEE Transactions on Information Forensics and Security*, *13*(5), 1333-1345.

- FL with additive HE for deep learning, no leakage

Bassily, R., Smith, A., & Thakurta, A. (2014, October). Private empirical risk minimization: Efficient algorithms and tight error bounds. In *2014 IEEE 55th Annual Symposium on Foundations of Computer Science* (pp. 464-473). IEEE.

- DP tf implementation reference of amortized epsilon

Beimel, A., Kasiviswanathan, S. P., & Nissim, K. (2010, February). Bounds on the sample complexity for private learning and private data release. In *Theory of Cryptography Conference* (pp. 437-454). Springer, Berlin, Heidelberg. 

- Privacy amplification via sampling

Bun, M., & Steinke, T. (2016, November). Concentrated differential privacy: Simplifications, extensions, and lower bounds. In *Theory of Cryptography Conference* (pp. 635-658). Springer, Berlin, Heidelberg.

- PATE, previous work related to moment accountant

Dwork, C., Rothblum, G. N., & Vadhan, S. (2010, October). Boosting and differential privacy. In *2010 IEEE 51st Annual Symposium on Foundations of Computer Science* (pp. 51-60). IEEE.

- Strong composition theorem

Dwork, C., & Roth, A. (2014). The algorithmic foundations of differential privacy. *Foundations and Trends® in Theoretical Computer Science*, *9*(3–4), 211-407.

- DP bible
- Advanced composition theorem

Dwork, C., & Rothblum, G. N. (2016). Concentrated differential privacy. *arXiv preprint arXiv:1603.01887*.

- PATE, previous work related to moment accountant

Goodfellow, I., Pouget-Abadie, J., Mirza, M., Xu, B., Warde-Farley, D., Ozair, S., ... & Bengio, Y. (2014). Generative adversarial nets. In *Advances in neural information processing systems* (pp. 2672-2680).

- GAN, used in PATE

Goryczka, S., Xiong, L., & Sunderam, V. (2013, March). Secure multiparty aggregation with differential privacy: A comparative study. In *Proceedings of the Joint EDBT/ICDT 2013 Workshops* (pp. 155-163).

- Comparison of perturbation based method, HE, SS, computation time

Kairouz, P., Oh, S., & Viswanath, P. (2014). Extremal mechanisms for local differential privacy. In *Advances in neural information processing systems* (pp. 2879-2887).

- Extreme mechanism, local DP

Nissim, K., Raskhodnikova, S., & Smith, A. (2007, June). Smooth sensitivity and sampling in private data analysis. In *Proceedings of the thirty-ninth annual ACM symposium on Theory of computing* (pp. 75-84).

- Smooth sensitivity in PATE

Papernot, N., Abadi, M., Erlingsson, U., Goodfellow, I., & Talwar, K. (2016). Semi-supervised knowledge transfer for deep learning from private training data. *arXiv preprint arXiv:1610.05755*.

- PATE
- GAN

Ryffel, T., Trask, A., Dahl, M., Wagner, B., Mancuso, J., Rueckert, D., & Passerat-Palmbach, J. (2018). A generic framework for privacy preserving deep learning. *arXiv preprint arXiv:1811.04017*.

- Syft

Salimans, T., Goodfellow, I., Zaremba, W., Cheung, V., Radford, A., & Chen, X. (2016). Improved techniques for training gans. In *Advances in neural information processing systems* (pp. 2234-2242).

- Improved GAN

Tang, X., Zhu, L., Shen, M., & Du, X. (2018). When Homomorphic Cryptosystem Meets Differential Privacy: Training Machine Learning Classifier with Privacy Protection. *arXiv preprint arXiv:1812.02292*.

- Comparison, DP vs. HE

Volgushev, N., Schwarzkopf, M., Getchell, B., Varia, M., Lapets, A., & Bestavros, A. (2019, March). Conclave: secure multi-party computation on big data. In *Proceedings of the Fourteenth EuroSys Conference 2019* (pp. 1-18).

- MPC on big data

Xu, R., Baracaldo, N., Zhou, Y., Anwar, A., & Ludwig, H. (2019, November). HybridAlpha: An Efficient Approach for Privacy-Preserving Federated Learning. In *Proceedings of the 12th ACM Workshop on Artificial Intelligence and Security* (pp. 13-23).

- HybridAlpha, comparison between different privacy preserving methods, SMC with HE, Local DP, SMC with no privacy

Yang, Q., Liu, Y., Cheng, Y., Kang, Y., Chen, T., & Yu, H. (2019). Federated learning. *Synthesis Lectures on Artificial Intelligence and Machine Learning*, *13*(3), 1-207.

- A good review book of FL







