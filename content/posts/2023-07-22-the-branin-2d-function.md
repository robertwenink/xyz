---
title: The Branin 2D function
date: 2023-06-22T07:18:06.254Z
featuredImage: /images/uploads/branin_transformed_alternating.png

tags: ["Test tag"]
categories: ["Test Category"]

draft: true

---

## The Branin 2D function

Does this really need text? Yes it does. Furthermore, heading1 is not included in the toc (probably with reason, the font size is larger than that of the title!)

### Test 2

***This is italic and bold text***

And this is a quote:

> Wowzers what an empty site

Lets insert some python code :


```python
class Encoder(json.JSONEncoder):
    """ Special json encoder for numpy types """
    def default(self, obj):
        if isinstance(obj, np.integer):
            return int(obj)
        elif isinstance(obj, np.floating):
            return float(obj)
        elif isinstance(obj, np.ndarray):
            return obj.tolist()
        return json.JSONEncoder.default(self, obj)
```

![test](/images/uploads/branin_transformed_alternating.png "The Branin 2d Function plotted inline")

#### Continuation

Let's continue with the post here after the image.

#### Extra

Even more text, at the same heading level

##### Even deeper

This is heading 5

###### Deepest 

And this is heading level 6. Everything from level 3 (2nd real level) and on is not included in the toc.

### Testing Latex

Following the principle of a Gaussian Process, it is typically assumed that data at locations $\mathbf{x}={x*0,...x_n}$ is the result of sampling a stochastic process $\boldsymbol{Y}(x) = \mu + \mathcal{N}(0,\sigma^2)$. We further assume realisations of $\boldsymbol{Y(x)}$ that are spatially near to each other are correlated. We describe this correlation using a covariance matrix $\boldsymbol{\Sigma}$, composed of an (unknown) process variance $\sigma$ and the correlation matrix $\boldsymbol{R}$:

\begin{equation}
\boldsymbol{\Sigma}\left(\boldsymbol{Y}\right)=\sigma^{2} \boldsymbol{R}
\end{equation}

To define the correlation matrix $\boldsymbol{R}$, \citet{Jones2001} and many others use the so-called "Kriging" kernel:

\begin{equation}
\mathbf{R}\left[\mathbf{Y}(x^{i}), \mathbf{Y}(x^{j})\right]=
\exp \left(-\sum_{\ell=1}^{d} \theta_{\ell}\left|\mathbf{x}^{i \ell}-\mathbf{x}^{j \ell}\right|^{p_{\ell}}\right)
\end{equation}

The end.