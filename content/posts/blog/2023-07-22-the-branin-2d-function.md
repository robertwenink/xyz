---
layout: blog
title: The Branin 2D function
date: 2023-07-22T07:18:06.254Z
thumbnail: /images/uploads/branin_transformed_alternating.png
rating: 1
---
## The Branin 2D function

D﻿oes this really need text? Yes it does. Furthermore, heading1 is not included in the toc (probably with reason, the font size is larger than that of the title!

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

A﻿nd this is heading level 6. The heading text is smaller than the actual text!

T﻿he end.