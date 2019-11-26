# DGA Intel

Using deep learning to detect DGA domains.

# Overview
The DGAIntel Python module allows you to utilize a powerful CNN-LSTM model to predict whether a given domain name was generated by a domain generation algorithm (DGA) or corresponds to a genuine domain. The prediction features are also accesible through [this website](http://www.dgaintel.com/), but this package allows for direct integration into your workflow.

## Requirements

DGAIntel is designed for use with Python 3. It has only two requirements:

    - TensorFlow 2.x
    - Numpy

# Installation

To download dgaintel, simply use Pypi via pip.
```sh
$ pip install dgaintel
```

Alternatively, you could install from source.
```sh
$ git clone https://github.com/sudo-rushil/dgaintel
$ cd dgaintel
$ python setup.py install
```

Verify your installation by running
```Python
>>> import dgaintel
>>> dgaintel.get_prediction('microsoft.com')
'microsoft.com is genuine with probability 0.00050'
```

# Examples

### Predict DGA
This is simple way of determining whether any given domain, such as `'microsoft.com'` is DGA or not, mainly intended for cyber security analysts.

```Python
from dgaintel import get_prediction

get_prediction('microsoft.com')
```
> 'microsoft.com is genuine with probability 0.00050'

### Predict DGA probability
This allows for getting the probability, or probabilities, that a domain or list of domains is DGA or not, which is more useful to data scientists.

```Python
from dgaintel import get_prob

# For single domain
prob = get_prob('microsoft.com')
print(prob)

# For multiple domains
probs = get_prob(['microsoft.com', 'wikipedia.com', 'vlurgpeddygdy.com'])
print(probs)

# To get just the scores
raw_probs = list(get_prob(['microsoft.com', 'wikipedia.com', 'vlurgpeddygdy.com']))
print(raw_probs)
```

> 0.00050

> [('microsoft.com', 0.00050), ('wikipedia.com', 0.00033), ('vlurgpeddygdy.com', 0.97601)]

> [0.00050845, 0.00033092, 0.00144754]

### Predict by file
This is for inputing a file containing a list of domains to get predictions on all of them at once, which is helpful for data analysts.

Say you have a domain file `domains.txt`.
```
microsoft.com
wikipedia.com
vlurgpeddygdy.com
```

Then, you can run the following code in the same directory.
```Python
from dgaintel import get_prediction

# Print to console
get_prediction('domains.txt')

# Write to file
get_prediction('domains.txt', to_file='domain_predictions.txt')
```

> microsoft.com is genuine with probability 0.00050

> wikipedia.com is genuine with probability 0.00033

> vlurgpeddygdy.com is DGA with probability 0.97601

If you read the new file `domain_predictions.txt`, you will see the following.

```
microsoft.com is genuine with probability 0.0005084535223431885
wikipedia.com is genuine with probability 0.00033092446392402053
vlurgpeddygdy.com is DGA with probability 0.9760094285011292
```

### Prediction analysis 
This is an example function that integrates dgaintel with [whois](https://pypi.org/project/whois/) for performing basic prediction analysis, which is important for cyber security investigators.

```Python
from dgaintel import get_prob
from whois import query

def analyze(domain, print=True):
    prob = get_prob(domain)
    whois = query(domain)
    dga = False
    if prob >= 0.5: dga = True

    domain_analysis = {'domain_name': domain, 'dga': dga, 'registrar': whois.registrar, 'creation date' : whois.creation_date, 'expiration date': whois.expiration_date}

    if print:
        print()
        for key, val in items(domain_analysis):
            print('{}: {}'.format(key, val))
        print()
        return None
    
    return domain_analysis

analyze('microsoft.com')

# Get analysis dictionary in python itself
analysis = analyze('microsoft.com', print=False)
```

> name: microsoft.com

> dga: False

> registrar: MarkMonitor Inc.

> creation date: 1991-05-02 04:00:00

> expiration date: 2021-05-03 04:00:00


# Documentation
DGAIntel has support for polymorphism; to input domains to run predictions on, you can use a single domain name, a list of domain names, or a text file with line-separated domain names. The text file has the format

```
microsoft.com
wikipedia.com
vlurgpeddygdy.com
...
```

Additionally, the Tensorflow Keras model running in the backend supports input batching, meaning there is a significant increase in speed for running predictions on lists or files rather than individual domains. This was tested in Jupyter.

```Python
from dgaintel import get_prob

# List of 10 domain names
l = ['microsoft.com', 'squarespace.com', 'hsfkjdshfjasdhfk.com', 'fdkhakshfda.com', 'foilfencersarebad.com', 'foilfencersarebad.com', 'foilfencersarebad.com', 'discojjfdsf.com', 'fasddafhkj.com', 'wikipedai.com']
```

```Python
# One domain
%%timeit
get_prob(l[0])
```

> 286 ms ± 4.99 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

```Python
# Ten domains
%%timeit
get_prob(l)
```

> 290 ms ± 7.23 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

```Python
# Hundred domains
%%timeit
get_prob(l*10)
```

> 333 ms ± 4.71 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

```Python
# Thousand domains
%%timeit
get_prob(l*100)
```

> 584 ms ± 14.8 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

This demonstrates that increasing the number of domain names one runs the prediction by 1000x only increases the inference time by less than 2x. Therefore, this model is easily adaptable to large-scale predictions.

## API 
The `get_prediction` function will either print the predictions or write them to a user-specified file.
```Python
from dgaintel import get_prediction

get_prediction('microsoft.com')
get_prediction(['microsoft.com', 'wikipedia.com', 'vlurgpeddygdy.com'])
get_prediction('domains.txt')
get_prediction('domains.txt', to_file='domain_predictions.txt')
```

The `get_prob` function will perform the inference and provide the prediction floats. It is helpful if you want to use the prediction scores directly in your workflow.
```Python
from dgaintel import get_prob

get_prob('microsoft.com') # 0.00050851
get_prob(['microsoft.com', 'wikipedia.com', 'vlurgpeddygdy.com']) # [('microsoft.com', 0.00050), ('wikipedia.com', 0.00033), ('vlurgpeddygdy.com', 0.0.97601)]
get_prob('domains.txt') # [('microsoft.com', 0.00050), ('wikipedia.com', 0.00033), ('vlurgpeddygdy.com', 0.97601)]
get_prob(['microsoft.com', 'wikipedia.com', 'google.com'], raw=True) # array([0.00050, 0.00033, 0.0.97601], dtype=float32)
```