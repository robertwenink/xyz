---
layout: blog
title: The Branin 2D function
date: 2023-06-22T07:18:06.254Z
lastmod: 2023-07-28T07:18:06.254Z
featuredImage: /images/uploads/branin_transformed_alternating.png

tags: ["Test tag"]
categories: ["Test Category"]

draft: false

toc:
  enable: true
  auto: true
---
## The Branin 2D function

D﻿oes this really need text? Yes it does. Furthermore, heading1 is not included in the toc (probably with reason, the font size is larger than that of the title!)

### T﻿est 2

***T﻿his is italic and bold text***

A﻿nd this is a quote:

> W﻿owzers what an empty site

L﻿ets insert some python code :


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

#### C﻿ontinuation

L﻿et's continue with the post here after the image.

#### E﻿xtra

E﻿ven more text, at the same heading level

##### E﻿ven deeper

T﻿his is heading 5

###### D﻿eepest 

A﻿nd this is heading level 6. Everything from level 3 (2nd real level) and on is not included in the toc.

### T﻿esting Latex

Following the principle of a Gaussian Process, it is typically assumed that data at locations $\mathbf{x}={x*0,...x_n}$ is the result of sampling a stochastic process $\boldsymbol{Y}(x) = \mu + \mathcal{N}(0,\sigma^2)$. We further assume realisations of $\boldsymbol{Y(x)}$ that are spatially near to each other are correlated. We describe this correlation using a covariance matrix $\boldsymbol{\Sigma}$, composed of an (unknown) process variance $\sigma$ and the correlation matrix $\boldsymbol{R}$:

\begin{equation}
\boldsymbol{\Sigma}\left(\boldsymbol{Y}\right)=\sigma^{2} \boldsymbol{R}
\end{equation}

To define the correlation matrix $\boldsymbol{R}$, \citet{Jones2001} and many others use the so-called "Kriging" kernel:

{{< raw >}}
\begin{equation}
\begin{aligned}
\boldsymbol{R}\left[\boldsymbol{Y}(x*{i}), \boldsymbol{Y}(x*{j})\right]=
\exp \left(-\sum*{\ell=1}^{d} \theta*{\ell}\left|\boldsymbol{x}*{i \ell}-\boldsymbol{x}*{j \ell}\right|^{p*{\ell}}\right)
\end{aligned}
\end{equation}
{{< /raw >}}
T﻿he end.