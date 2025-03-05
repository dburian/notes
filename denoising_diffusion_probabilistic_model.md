---
tags: [ml, generative]
---
# DDPM

Denoising Diffusion Probabilistic Model (*DDPM*) is a [Diffusion Probabilistic
Model (*DPM*)](./diffusion_probabilistic_model.md) model introduced by [Ho et
al. (2020)](https://arxiv.org/pdf/2006.11239).

DDPM model follows the high-level intuition from
[DPM](./diffusion_probabilistic_model.md), but the authors connect DPM with
score-based models (TODO: note), while being able to produce much better looking
images - basically fine-tuning the method.

## Training

In this section I go over the DDPM's training. First, let me go over the main
ideas quickly, then I'll drill into the math and details in the following
subsections.

### Forward and Reverse processes

Same as [DPM](./diffusion_probabilistic_model.md), DDPM is trained by reversing
the Markovian **forward process**, which adds noise to a training input $x_0$,
through steps $t = 1\ldots T$:

$$
q(\mathbf{x}_t | \mathbf{x}_{t-1}) = \mathcal{N}\left(
  \mathbf{x}_t;
  \sqrt{\beta_t - 1}\mathbf{x}_{t-1},
  \beta_t\mathbf{I}
\right)
$$

The goal of the model is to approximate **the reverse process**:

$$
p_\theta (\mathbf{x}_{t - 1} | \mathbf{x}_t) = \mathcal{N}\left(
  \mathbf{x}_{t - 1};
  \mu_\theta(\mathbf{x}_t, t),
  \Sigma_\theta(\mathbf{x}_t, t)
\right)
$$

### ELBO loss

We train $p_\theta$ through variational lower bound (also called [Evidence Lower
Bound (ELBO)](./elbo.md)):

$$
\begin{aligned}
-\mathbb{E}_{q(\mathbf{x}_0)}[\log p_\theta(\mathbf{x}_0)] &=
  -\mathbb{E}_{q(\mathbf{x}_0)}[\log\mathbb{E}_{p_\theta(\mathbf{x}_{1:T})}[p_\theta(\mathbf{x}_0|\mathbf{x}_{1:T})]] \\
&\leq -\mathbb{E}_{q(\mathbf{x}_{0:T})}\left[\log\frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}\right] \\
&
  = \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[
    \underbrace{
      D_{KL}(q(\mathbf{x}_T|\mathbf{x}_0)\|p_\theta(\mathbf{x}_T))
    }_{L_T}
    + \sum_{t=2}^T \underbrace{
        D_{KL}(q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)\|p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t))
      }_{L_t}
    - \underbrace{
      \log p_\theta(\mathbf{x}_0|\mathbf{x}_1)
    }_{L_0}
  \right]
\end{aligned}
$$

Since the authors fix forward process, we can leave $L_T$ out when training.
$L_0$ deals with 'decoding' real-valued latents into natural numbers, which
isn't that much important and so I don't elaborate on it now. Instead, let's
focus on $L_t$.

The authors fix $\Sigma_\theta(\mathbf{x}_t, t) := \sigma_t
\mathbf{I}$, and only train $\mu_\theta(\mathbf{x}_t, t)$. Eliminating untrained
variables we get:

$$
L_{t-1} = \mathbb{E}_q\left[
  \frac{1}{2\sigma_t^2}
  ||
    \tilde{\mu}_t(\mathbf{x}_t, \mathbf{x}_0) -
    \mu_\theta(\mathbf{x}_t, t)
  ||^2
\right] + C,
$$

where $\tilde{\mu}_t$ is true mean derived from $q$. By reframing the model, to
model the noise in $x_{t+1}$ labelled as $\mathbf{\epsilon}$, instead of the
un-noised mean $\mu_t$, we can even write $L_{t-1}$ as:

$$
L_{t-1} = \mathbb{E}_{\mathbf{x}_t, \mathbf{\epsilon}} \left[
  \frac{
    \beta_t^2
  }{
    2\sigma_t^2\alpha_t(1 - \bar{\alpha}_t)
  }
  ||
    \mathbf{\epsilon} -
    \mathbf{\epsilon}_\theta(\mathbf{x}_t, t)
  ||^2
\right]
$$

The authors found that training with simplified loss $L_{\text{simple}}$, to be
beneficial for sample quality:

$$
L_{\text{simple}}(\theta) :=
  \mathbb{E}_{t, \mathbf{x}_t, \mathbf{\epsilon}} \left[
    ||
      \mathbf{\epsilon} -
      \mathbf{\epsilon}_\theta(\mathbf{x}_t, t)
    ||^2
  \right]
$$

### Training algorithm

To train the reverse process the authors:

1. get a training sample $\mathbf{x}_0$
2. noise it with forward process by doing $t \sim \text{Uniform}(\{1, \ldots,
   T\})$ steps to obtain $\mathbf{x}_t$
3. train the model to predict the noise that was added to $\mathbf{x}_0$ from
   $\mathbf{x}_t$ with $L_\text{simple}$.

There is a trick, with which you don't have to iterate over the forward process
$t$ times and instead get $\mathbf{x}_t$ in constant time.

### Sampling

The sampling algorithm:
1. takes random noise $\mathbf{x}_T \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$
2. iteratively sample from the reverse's step distribution:
    1. predict noise in $\mathbf{x}_t$ and with it predict the mean of $\mathbf{x}_{t-1}$
    2. add scaled variance to the mean to obtain $\mathbf{x}_{t-1}$

The iterating over the reverse process is the slow part since we need to do a
forward pass through network $T$ times (in experiments the authors used $T =
1000$). This is an efficiency problem that the subsequent papers (e.g. [Improved
DDPM](./improved_ddpm.md)) solve.

### Summary

The authors train a model to approximate *reverse process* of a
__fixed__ markovian *forward process*, that introduces noise to the input. The
model is trained on variational lower bound, after __freezing the variance in
the reverse process__ and reframing the model's prediction as noise part of the
input. Moreover, the authors propose __re-weighted__ version of variational
lower bound, which leads to improved sample quality.

## Training details

Now we go over the details.

### Forward process

The forward process $q$ is expressed as a normal distribution with std. deviation
$\beta_t$:

$$
\begin{aligned}
q(\mathbf{x}_t | \mathbf{x}_{t-1}) &= \mathcal{N}\left(
  \mathbf{x}_t;
  \sqrt{\beta_t - 1}\mathbf{x}_{t-1},
  \beta_t\mathbf{I}
\right) \\
&= \sqrt{\beta_t - 1} \mathbf{x}_{t-1} + \mathbf{e}\sqrt{\beta_t}
  \text{ for } \mathbf{e} \sim \mathcal{N}(0, \mathbf{I})
\end{aligned}
$$

We can express $q$ even after $t$ steps of forward process (not just one):

$$
\begin{aligned}
\mathbf{x}_t
&= \sqrt{\alpha_t}\mathbf{x}_{t-1} + \sqrt{1-\alpha_t}\mathbf{e}_t \\
&= \sqrt{\alpha_t}\left(\sqrt{\alpha_{t-1}}\mathbf{x}_{t-2} +
  \sqrt{1-\alpha_{t-1}}\mathbf{e}_{t-1}\right) + \sqrt{1-\alpha_t}\mathbf{e}_t \\
&= \sqrt{\alpha_t\alpha_{t-1}}\mathbf{x}_{t-2} +
  \sqrt{\alpha_t(1-\alpha_{t-1}) + (1-\alpha_t)}\mathbf{\bar{e}}_{t-1} \\
\end{aligned}
$$

since [the sum of normally distributed variables is also a normally distributed
variable with variance equal to sum of variables'
variances](./normal_distribution.md). Continuing:

$$
\begin{aligned}
&= \sqrt{\alpha_t\alpha_{t-1}}\mathbf{x}_{t-2} +
  \sqrt{1-\alpha_t\alpha_{t-1}}\mathbf{\bar{e}}_{t-1} \\
&= \sqrt{\alpha_t\alpha_{t-1}\alpha_{t-2}}\mathbf{x}_{t-3} +
  \sqrt{1-\alpha_t\alpha_{t-1}\alpha_{t-2}}\mathbf{\bar{e}}_{t-2} \\
&= \cdots \\
&= \sqrt{\bar{\alpha_t}}\mathbf{x}_0 + \sqrt{1-\bar{\alpha_t}}\mathbf{\bar{e}}_0,
\end{aligned}
$$

where:
- $\alpha_t = 1-\beta_t$
- $\bar{\alpha_t} = \prod_{i=1}^t \alpha_i$, and
- $\mathbf{e}_t, \mathbf{\bar{e}}_t \sim \mathcal{N}(0, \mathbf{I})$:

So even after $t$ steps, $q$ is **normally distributed**. This allows us to
define $x_t$ w.r.t. $x_0$:

$$
q(\mathbf{x}_t | \mathbf{x}_0) = \mathcal{N}\left(
  \sqrt{\bar{\alpha_t}}\mathbf{x}_0,
  1-\bar{\alpha_t} \mathbf{I}
\right)
$$

Moreover, if $\bar{\alpha_t} \rightarrow 0$ as $t \rightarrow \inf$ then $x_t \sim
\mathcal{N}(0, \mathbf{I})$. This property will be handy when sampling, since we
can just start with $x_T \sim \mathcal{N}(0, \mathbf{I})$ and the model should
gradually de-noise it to something similar to a training sample. 

We could have labelled mean and variance in any way. The thing to notice is that
their definitions are tied. And only thanks to these tied definitions,
$q(\mathbf{x}_T | \mathbf{x}_0)$ can be defined so simply and has the above
mentioned properties.

### Reverse process

If $\beta_t$ is sufficiently small the reverse of $q(x_t | x_{t-1})$, which we
label as $p_\theta$ is also a normal distribution (proven by [Feller
(1949)](https://projecteuclid.org/ebooks/berkeley-symposium-on-mathematical-statistics-and-probability/On-the-Theory-of-Stochastic-Processes-with-Particular-Reference-to/chapter/On-the-Theory-of-Stocha%20stic-Processes-with-Particular-Reference-to/bsmsp/1166219215)):

$$
p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t) =
\mathcal{N}(
  \mathbf{x}_{t-1};
  \mu_\theta(\mathbf{x}_t,t),
  \sigma^2_t\mathbf{I}
)
$$

#### The true reverse process

So essentially we need to find the correct values for $\mu_\theta$ and
$\sigma_t$. We do so, by trying to find reverse of $q(x_t | x_{t-1})$. However,
this reverse is only tractable if we condition it on the original sample $x_0$:


$$
\begin{align}
q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) &=
  q(\mathbf{x}_{t-1}, \mathbf{x}_t | \mathbf{x}_0)
  \frac{1}{q(\mathbf{x}_t | \mathbf{x}_0)} \\
&= q(\mathbf{x}_t | \mathbf{x}_{t-1}, \mathbf{x}_0)
\frac{
    q(\mathbf{x}_{t-1} | \mathbf{x}_0)
  }{
    q(\mathbf{x}_t | \mathbf{x}_0)
  }\\
&= q(\mathbf{x}_t | \mathbf{x}_{t-1})
\frac{
    q(\mathbf{x}_{t-1} | \mathbf{x}_0)
  }{
    q(\mathbf{x}_t | \mathbf{x}_0)
  }\\
\end{align}
$$

These are just normal distributions we defined above, where the last equation
holds since $q(x_0, x_t | x_{t-1}) = q(x_t | x_{t-1})$ ($q(x_t)$ is either
defined using $x_{t-1}$ or $x_0$, both are not needed). Expending the definition
of [Normal distribution](./normal_distribution.md) and leaving out the
normalization constant we get:

$$
\begin{align}
q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) &=
 \textcolor{purple}{q(\mathbf{x}_t | \mathbf{x}_{t-1})}
  \frac{
    \textcolor{red}{q(\mathbf{x}_{t-1} | \mathbf{x}_0)}
  }{
    \textcolor{green}{q(\mathbf{x}_t | \mathbf{x}_0)}
  }\\
&\propto \exp \left(-\frac{1}{2}\left[
  \textcolor{purple}{
    \frac{
      (\mathbf{x}_t-\sqrt{\alpha_t}\mathbf{x}_{t-1})^2
    }{
      \beta_t
    }
  } + \textcolor{red}{
    \frac{
      (\mathbf{x}_{t-1}-\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_0)^2
    }{
      1-\bar{\alpha}_{t-1}
    }
  } - \textcolor{green}{
    \frac{
      (\mathbf{x}_t-\sqrt{\bar{\alpha}_t}\mathbf{x}_0)^2
    }{
      1-\bar{\alpha}_t
    }
  }
\right]\right)\\
&= \exp\left(-\frac{1}{2}\left[
      \frac{
        \mathbf{x}_t^2-
        2\sqrt{\alpha_t}\mathbf{x}_t\mathbf{x}_{t-1}+
        \alpha_t\mathbf{x}_{t-1}^2
      }{
        \beta_t
      } +
      \frac{
        \mathbf{x}_{t-1}^2-2\sqrt{\bar{\alpha}_{t-1}}\mathbf{x}_{t-1}\mathbf{x}_0+\bar{\alpha}_{t-1}\mathbf{x}_0^2
      }{
        1-\bar{\alpha}_{t-1}
      } +
      C(\mathbf{x}_t, \mathbf{x}_0)
  \right]
\right)\\
&\propto \exp\left(-\frac{1}{2}\left[
    \left(
      \frac{\alpha_t}{\beta_t} +
      \frac{1}{1-\bar{\alpha}_{t-1}}
    \right) \mathbf{x}_{t-1}^2 -
    2\left(
      \frac{ \sqrt{\alpha_t}}{\beta_t}\mathbf{x}_t +
      \frac{\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}\mathbf{x}_0
    \right)\mathbf{x}_{t-1}
  \right]
\right) \\
&= \exp\left(
  -\frac{1}{2}
  \frac{
    \alpha_t(1 - \bar{\alpha}_{t-1}) + 1 - \alpha_t
  }{
    (1 - \alpha_t)(1 - \bar{\alpha}_{t-1})
  }
  \left[
    \mathbf{x}^2_{t-1}
    - 2\left(
      \frac{ \sqrt{\alpha_t}}{\beta_t}\mathbf{x}_t +
      \frac{\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}\mathbf{x}_0
    \right) \frac{1}{
      \frac{
        \alpha_t(1 - \bar{\alpha}_{t-1}) + 1 - \alpha_t
      }{
        (1 - \alpha_t)(1 - \bar{\alpha}_{t-1})
      }
    } \mathbf{x}_{t-1}
  \right]
\right) \\
&\propto \mathcal{N}\left(
  \mathbf{x}_{t-1};
  \frac{
    \left(
      \frac{ \sqrt{\alpha_t}}{\beta_t}\mathbf{x}_t +
      \frac{\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}\mathbf{x}_0
    \right)
    }{
      \frac{
        \alpha_t(1 - \bar{\alpha}_{t-1}) + 1 - \alpha_t
      }{
        (1 - \alpha_t)(1 - \bar{\alpha}_{t-1})
      }
    }
  ,
  \frac{
    (1 - \alpha_t)(1 - \bar{\alpha}_{t-1})
  }{
    \alpha_t(1 - \bar{\alpha}_{t-1}) + 1 - \alpha_t
  } \mathbf{I}
\right)
\end{align}
$$

Where the reasons for the proportional symbols are (in the same order):
1. Leaving out normalization of the function (so it sums to one)
2. Leaving out term $C(\mathbf{x}_t, \mathbf{x}_0)$, which is constant to
   $\mathbf{x}_{t-1}$
3. Introducing the term $C(\mathbf{x}_t, \mathbf{x}_0)$ back to complete the
   square in brackets and the normalization so that the function sums to one

Labelling the mean $\mathbf{\tilde{\mu}}_\theta(\mathbf{x}_t, \mathbf{x}_0)$ and
variance $\tilde{\beta}_t$ we have:

$$
\tilde{\beta}_t =
  \frac{
    (1 - \alpha_t)(1 - \bar{\alpha}_{t-1})
  }{
    \alpha_t(1 - \bar{\alpha}_{t-1}) + 1 - \alpha_t
  }
= \frac{
    1 - \bar{\alpha}_{t-1}
  }{
    \alpha_t - \bar{\alpha}_{t} + 1 - \alpha_t
  }\beta_t
= \frac{
    1 - \bar{\alpha}_{t-1}
  }{
    1 - \bar{\alpha}_{t}
  }\beta_t
$$

$$
\mathbf{\tilde{\mu}}_\theta(\mathbf{x}_t, \mathbf{x}_0) =
  \frac{
    \left(
      \frac{
        \sqrt{\alpha_t}
      }{
        1 - \alpha_t
      }\mathbf{x}_t +
      \frac{
        \sqrt{\bar{\alpha}_{t-1}}
      }{
        1-\bar{\alpha}_{t-1}
      }\mathbf{x}_0
    \right)
        (1 - \alpha_t)(1 - \bar{\alpha}_{t-1})
    }{
        1 - \bar{\alpha}_{t}
    }
= \frac{
    \sqrt{\alpha_t}(1 - \bar{\alpha}_{t-1})\mathbf{x}_t +
    \sqrt{\bar{\alpha}_{t-1}}(1 - \alpha_t)\mathbf{x}_0
    }{
        1 - \bar{\alpha}_{t}
    }
$$

And so: $q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0) =
\mathcal{N}\left(\mathbf{x}_{t-1}; \mathbf{\tilde{\mu}_\theta}(\mathbf{x}_t, \mathbf{x}_0),
\tilde{\beta_t}\mathbf{I}\right)$.

#### Reverse process variance

For the reverse process $p_\theta$, authors choose to fix
$\mathbf{\Sigma_\theta}$ to a diagonal matrix filled with time-dependent
constants $\sigma_t$ (i.e. they are not trained).

The authors experimented with training $\sigma^2_\theta(t)$ (time-independent
model), but the training turned out to be very unstable (unable to converge).

There are two extremes for $\sigma_t$, which correspond to reverse step's
[entropy](./entropy.md) lower and upper bound found in [DPM's
paper](./diffusion_probabilistic_model.md):

- **lower boundary** corresponding to $H_q(\mathbf{X}_{t-1} | \mathbf{X}_t,
  \mathbf{X}_0)$ and stemming from the definition of $q(\mathbf{x}_{t-1} |
  \mathbf{x}_t, \mathbf{x}_0)$ above:

$$
\sigma^2_t = \tilde{\beta} = \frac{
    1 - \bar{\alpha}_{t-1}
  }{
    1 - \bar{\alpha}_{t}
  }\beta_t
$$

- **upper boundary** corresponding to $H_q(\mathbf{X}_t | \mathbf{X}_{t-1})$ and
  stemming from the definition of the forward process $q(\mathbf{x}_t |
  \mathbf{x}_{t-1})$:

$$
\sigma^2_t = \beta
$$

If we use the sigma corresponding to the lower boundary, the reverse process
could arrive to a single $\mathbf{x}_0$, assuming it was given. On the other
hand, if we use the sigma corresponding to the upper boundary, the reverse
process would do exactly the same as the forward one and would give use
$\mathcal{N}(0, \mathbf{I})$. So depending on the entropy of $\mathbf{X}_0$ we
want to optimize for, we can choose $\sigma_t$.

In the experiments, the authors found that **both of these extremes give similar
results**, and so they (AFAIK) didn't experimented with this hyperparameter
further.

### Loss

To train $\mathbf{\mu_\theta}$ and $\mathbf{\Sigma_\theta}$ we use
[ELBO](./elbo.md) (which uses [Jensen's
inequality](https://en.wikipedia.org/wiki/Jensen's_inequality) for the
approximation):

$$
\begin{aligned}
-\mathbb{E}_{q(\mathbf{x}_0)}\left[
  \log p_\theta(\mathbf{x}_0)
\right]
&= -\mathbb{E}_{q(\mathbf{x}_0)}\left[
  \log\mathbb{E}_{p_\theta(\mathbf{x}_{1:T})}
    p_\theta(\mathbf{x}_0|\mathbf{x}_{1:T})
\right] \\
&= -\mathbb{E}_{q(\mathbf{x}_0)}\left[
  \log\mathbb{E}_{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}
    \frac{p_\theta(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T}|\mathbf{x}_0)}
\right] \\
&\leq -\mathbb{E}_{q(\mathbf{x}_{0:T})}\left[
  \log\frac{
    p_\theta(\mathbf{x}_{0:T})
  }{
    q(\mathbf{x}_{1:T}|\mathbf{x}_0)
  }
\right]\\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[
  \log\frac{
    \prod_{t=1}^T q(\mathbf{x}_t|\mathbf{x}_{t-1})
  }{
    p_\theta(\mathbf{x}_T)
    \prod_{t=1}^T p_\theta(\mathbf{x}_{t-1} | \mathbf{x}_t)
  }
\right]\\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[
  \log\frac{
    q(\mathbf{x}_{1:T}|\mathbf{x}_0)
  }{
    p_\theta(\mathbf{x}_{0:T})
  }
\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[
  -\log p_\theta(\mathbf{x}_T) +
  \sum_{t=2}^T \log\frac{
    q(\mathbf{x}_t|\mathbf{x}_{t-1})
  }{
    p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)
  } +
  \log\frac{
    q(\mathbf{x}_1|\mathbf{x}_0)
  }{
    p_\theta(\mathbf{x}_0|\mathbf{x}_1)
  }
\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[
  -\log p_\theta(\mathbf{x}_T) +
  \sum_{t=2}^T\log\left(
    \frac{
      q(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)
    }{
      p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)
    } \frac{
      q(\mathbf{x}_t|\mathbf{x}_0)
    }{
      q(\mathbf{x}_{t-1}|\mathbf{x}_0)
    }
  \right) +
  \log\frac{
    q(\mathbf{x}_1|\mathbf{x}_0)
  }{
    p_\theta(\mathbf{x}_0|\mathbf{x}_1)
  }
\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[
  -\log p_\theta(\mathbf{x}_T) +
  \sum_{t=2}^T\log\frac{
    q(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)
  }{
    p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)
  } + \log\frac{
    q(\mathbf{x}_T|\mathbf{x}_0)
  }{
    q(\mathbf{x}_1|\mathbf{x}_0)
  } + \log\frac{
    q(\mathbf{x}_1|\mathbf{x}_0)
  }{
    p_\theta(\mathbf{x}_0|\mathbf{x}_1)
  }
\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[
  \log\frac{
    q(\mathbf{x}_T|\mathbf{x}_0)
  }{
    p_\theta(\mathbf{x}_T)
  } +
  \sum_{t=2}^T\log\frac{
    q(\mathbf{x}_{t-1}|\mathbf{x}_t,\mathbf{x}_0)
  }{
    p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)
  } - \log p_\theta(\mathbf{x}_0|\mathbf{x}_1)
\right] \\
&= \mathbb{E}_{q(\mathbf{x}_{0:T})}\left[
  \underbrace{
    D_{KL}(q(\mathbf{x}_T|\mathbf{x}_0)\|p_\theta(\mathbf{x}_T))
  }_{L_T} +
  \sum_{t=2}^T\underbrace{
    D_{KL}(
      q(\mathbf{x}_{t-1}|\mathbf{x}_t, \mathbf{x}_0)\|
      p_\theta(\mathbf{x}_{t-1}|\mathbf{x}_t)
    )
  }_{L_t} -
  \underbrace{
    \log p_\theta(\mathbf{x}_0|\mathbf{x}_1)
  }_{L_0}\right]
\end{aligned}
$$

From these terms:

- $L_T$ is constant w.r.t. to $\theta$, since $q$ is fixed, and $p_\theta(x_T) = \mathcal{N}(0, \mathbf{I})$
- $L_0$ trains decoding $\mathbf{x}_1$ to $\mathbf{x}_0$
- $L_t$ trains the reverse process step

The decoding loss is a multidimensional Gaussian with diagonal covariance matrix
$\sigma^2_1\mathbf{I}$ centered around the true $\mathbf{x}_0$. All it does is
giving us probability of a discrete output $\mathbf{x}_0 \in \{0, 1, \ldots,
255\}$ given continuous $\mathbf{x}_1 \in [-1, 1]$.

#### Simplifying $L_{1:T-1}$

Since both the forward and reverse process are Gaussian, and the reverse
process's variance is non-trainable we can rewrite $L_{t-1}$ as:

$$
L_{t-1} = \mathbb{E}_q\left[
  \frac{1}{2\sigma^2_t} \|
    \tilde{\mathbf{\mu}}_t(\mathbf{x}_t, \mathbf{x}_0) -
    \mathbf{\mu}_\theta(\mathbf{x}_t, t)
  \|^2
\right] + C
$$

By using definition of $\mathbf{x}_t$ using $\mathbf{x}_0$:

$$
\mathbf{x}_t = \sqrt{\bar{\alpha}_t} x_0 + \sqrt{1 - \bar{\alpha}_t}\mathbf{e},
$$

and the definition of $\tilde{\mathbf{\mu}}_t(\mathbf{x}_t, \mathbf{x}_0)$, we
get:

$$
\begin{align}
\tilde{\mathbf{\mu}}_t(\mathbf{x}_t, \mathbf{x}_0) &=
  \frac{
    \sqrt{\alpha_t}(1 - \bar{\alpha}_{t-1})\mathbf{x}_t +
    \sqrt{\bar{\alpha}_{t-1}}(1 - \alpha_t)\mathbf{x}_0
    }{
        1 - \bar{\alpha}_{t}
    }\\
&= \frac{
    \sqrt{\alpha_t}(1 - \bar{\alpha}_{t-1})\mathbf{x}_t +
    \sqrt{\bar{\alpha}_{t-1}}(1 - \alpha_t)\left(
      \mathbf{x}_t - \sqrt{1 - \bar{\alpha}_t}\mathbf{e}
    \right) \frac{1}{\sqrt{\bar{\alpha}_t}}
    }{
        1 - \bar{\alpha}_{t}
    }\\
&= \ldots \\
&= \frac{1}{\sqrt{\alpha_t}} \left(
  \mathbf{x}_t - \frac{\beta_t}{\sqrt{1 - \bar{\alpha}_t}}\mathbf{e}
\right)
\end{align}
$$

Since $\mathbf{\mu}_\theta$ is given $\mathbf{x}_t$ as a parameter, it's only
job is to predict $\mathbf{e}$ given $\mathbf{x}_t$. So we parametrize
$\mathbf{\mu}_\theta$ as:
$$
\mathbf{\mu}_\theta(\mathbf{x}_t, t) = \frac{1}{\sqrt{\alpha_t}} \left(
  \mathbf{x}_t - \frac{\beta_t}{\sqrt{1 - \bar{\alpha}_t}}
  \mathbf{e}_\theta(\mathbf{x}_t, t)
\right)
$$

This gives us

$$
L_{t-1} - C = \mathbb{E}_{\mathbf{x}_0, \mathbf{e}} \left[
 \frac{\beta^2_t}{2\sigma^2_t\alpha_t(1-\bar{\alpha}_t)}
 \|
    \mathbf{e} -
    \mathbf{e}_\theta(
      \sqrt{\bar{\alpha}_t}\mathbf{x}_0 + \sqrt{1 - \bar{\alpha}_t}\mathbf{e},
      t
    )
  \|^2
\right]
$$

$L_{\text{simple}}$ just tosses the weighting:

$$
L_{\text{simple}} = \mathbb{E}_{\mathbf{x}_0, \mathbf{e}} \left[
 \|
    \mathbf{e} -
    \mathbf{e}_\theta(
      \sqrt{\bar{\alpha}_t}\mathbf{x}_0 + \sqrt{1 - \bar{\alpha}_t}\mathbf{e},
      t
    )
  \|^2
\right]
$$

When using $L_\text{simple}$ authors uniformly sample $t \in {1, \ldots, T}$. If
$t = 1$, $L_0$ without weighting is used instead of $L_\text{simple}$.

### Training algorithm

With the above simplified losses the training algorithm is straightforward:

1. get training sample $\mathbf{x}_0 \sim q(\mathbf{x}_0)$
2. get number of steps $t \sim \text{Uniform}(\{1, \ldots, T\})$
3. get random noise $\mathbf{e} \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$
4. Compute $L_\text{simple}$ and do a gradient step:

$$
\|
\mathbf{e} - \mathbf{e}_\theta(
  \sqrt{\bar{\alpha}_t}\mathbf{x}_0 +
  \sqrt{1-\bar{\alpha}_t}\mathbf{e},
  t
)
\|^2
$$

### Sampling

When sampling we need to sample from the reverse process $T$ times:

1. get random noise $\mathbf{x}_T \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$
2. for $T$ steps do:
    1. $\mathbf{z} \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$ if $t > 1$ else
       $\mathbf{z} = \mathbf{0}$
    2. sample from reverse process:

$$
\begin{align}
\mathbf{x}_{t-1} &= \mathbf{\mu}_\theta(\mathbf{x}_t, t) + \sigma_t \mathbf{z}\\
&= \frac{1}{\sqrt{\alpha_t}} \left(
  \mathbf{x}_t -
  \frac{1 - \alpha_t}{\sqrt{1 - \bar{\alpha}_t}} \mathbf{e}_\theta(\mathbf{x}_t, t)
\right) + \sigma_t \mathbf{z}\\
\end{align}
$$

## Network

The authors used a [U-network](./u_network.md) called PixelCNN++ (TODO) with
group normalization throughout it. The $t$ is embedded using [Transformer's
sinusoidal embeddings](./transformer.md).

## Experiments

For the experiments the authors used linear $\beta$ schedule $\beta_1 = 10^{-4}$
to $\beta_T = 0.02$. The aim was to keep $\beta_t$ small as possible (so that
reverse process is also a Gaussian) but large as possible so that we don't have
to do a lots of steps to arrive to a random noise. Keep in mind that betas are
increasing linearly, but the amount of noise is exponential in $t$ (viz
$\mathbf{x}_t$ expressed using $\mathbf{x}_0$).

During the experiments the authors found out that:

1. Learning diagonal $\mathbf{\Sigma}_\theta$ was unstable even when trained
   with full ELBO loss.

2. Training $\mathbf{\mu}_\theta$ was unstable with $L_\text{simple}$ and
   demanded training with full ELBO loss (i.e. the weighting in the loss was
   needed for the training to be stable).

3. Training the model to predict noise $\mathbf{e}_\theta$ results in similar
   quality as when training $\mathbf{\mu}_\theta$ when using full ELBO. With the
   simplified objective, training the model to predict the noise turned out to
   produce the best outputs.

### NLL/Bits per dimension/codelengths

The authors talk about NLL (negative log likelihood) as if the model could
generate one, which is confusing, because it clearly doesn't (otherwise we
wouldn't need ELBO). It turns out that when the authors state facts as *"...our
lossless codelengths are better than..."* they mean lower boundary of
NLL/[lossless codelengths](./lossless_codelengths.md) that stems from ELBO. I
deduced it from their
[implementation](https://github.com/hojonathanho/diffusion/blob/1e0dceb3b3495bbe19116a5e1b3596cd0706c543/diffusion_tf/diffusion_utils_2.py#L313).
I don't know why they don't clearly state its a lower bound, not the actual
likelihood.

TODO: FID

---
### Helpful sources

Apart from the paper itself there are many sources which explain it:

- [High-level Medium
  article by Gabriel Mongaras](https://medium.com/better-programming/diffusion-models-ddpms-ddims-and-classifier-free-guidance-e07b297b2869)
- [More involved article at AISummer](https://theaisummer.com/diffusion-models/)
- [Very involved paper written by Calvin Luo](https://arxiv.org/pdf/2208.11970)
- [Blog post with interactive
  visualizations](https://erdem.pl/2023/11/step-by-step-visual-introduction-to-diffusion-models#forward-diffusion-diagram)

