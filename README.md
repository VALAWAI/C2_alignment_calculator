# C2 Alignment Calculator

The Alignment Calculator is a C2 VALAWAI component that computes the alignment
if a set of norms or (*normative system*) with respect to a value of interest
using simulation and sampling.

The Alignment calculator works with an arbitrary *model*, which in practise is
implemented as a class. We denote the *state* of the model by $\mathbf{s}$. The
state $\mathbf{s}$ transitions at every time step according to the set of norms
$N=\{n_i\}$ in place. We denote a norm-regulated transition by $\mathbf{s}
\xrightarrow{N} \mathbf{s'}$. A succession of transitions $[\mathbf{s}_0
\xrightarrow{N} \mathbf{s}_1, ..., \mathbf{s}_{f-1} \xrightarrow{N}
\mathbf{s}_{f}]$ constitutes a *path*. The last state in a path $\mathbf{s}_f$
is an *outcome*.

To evaluate whether some set of norms $N$ is aligned with respect to a value $v$
or not, value $v$ has to be *grounded* in the context of the model we are
working with. This is done through the *value semantics function* $f_v:
\mathcal{S} \rightarrow [-1, 1]$, which maps the state of the model (i.e. the
object) to a number. The value semantics function evaluates the outcomes that
the norms $N$ steer the system towards. When this evaluation is positive, we
state that the norms $N$ *are aligned with respect to value $v$*. Formally:

$$
    \mathsf{Algn}_{N,v} = \mathbb{E}_N \left[f_v(\mathbf{S}_f)\right]
$$

The above states that the alignment $\mathsf{Algn}_{N,v}$ of the norms in $N$
with respect to value $v$ is computed as the expectation of the value semantics
function applied to the set of outcomes $\mathbf{S}_f$ under the implementation
of norms $N$ in the system. In practice, this expectation is computed by
simulating the evolution of the model under $N$ for a fixed number of time steps
(the *path length*), sampling their outcomes and evaluating them.

# Summary

 - Type: C2
 - Name: Alignment Calculator
 - Version: 1.0.0 (September 27, 2023)
 - API: [1.0.0 (February 3, 2023)](https://editor-next.swagger.io/?url=https://raw.githubusercontent.com/VALAWAI/C2_alignment_calculator/main/component-api.yml)
 - VALAWAI API: [0.1.0 (September 18, 2023)](https://editor-next.swagger.io/?url=https://raw.githubusercontent.com/VALAWAI/MOV/main/valawai-api.yml)
 - Developed by: IIIA-CSIC
 - License: [MIT](LICENSE)

# Usage

To understand how to set up and use this component, the reader is directed to
the `example.py` script available in this repository. There, you can find first
the `TaxModel` class. This class defined the model where norms are implemented
and where values are evaluated. The `TaxModel` class has a `step(norms)` method,
that takes in a `norms` dictionary and advances the state of the model by one
time step. The `norms` dictionary has a structure analogous to the following:

```python
{
    'pay': {
        'r0': 0.1,
        'r1': 0.2,
        'r2': 0.3,
        'r3': 0.4,
        'r4': 0.5
    },
    'payback': {
        'r0': 0.2,
        'r1': 0.2,
        'r2': 0.2,
        'r3': 0.2,
        'r4': 0.2
    }
}
```

The above norms dictionary indicates that there are two norms: `pay` (related to
the rates of tax payment) and `payback` (related to the rates of tax refunds).
Each norm is tied to a set of parameters, stating what is the rate of tax
payment or refund according to the wealth group of each agent in the model.

Next, the `example.py` script has two *value sematics functions* defined. These
are `ratio_wealth_value` and `gini_index_value`. Implemented value semantics
functions always take as input an instance of the model and return a float.

The Alignment Calculator component is implemented as a
[Flask](https://flask.palletsprojects.com/en/2.3.x/) application. To initialize
it, use the `create_app` function:

```python
from app import create_app

app = create_app(
    YourModel,                      # your model class
    [...],                          # your model initialization arguments
    {...},                          # your model initialization keyword arguments
    norms,                          # your norms dictionary
    your_value_semantics_funcion,   # your value semantics function
    path_length=10,                 # change if needed, default is 10
    path_sample=500                 # change if needed, default is 500
)
```

To develop your own model to be deployed in a component, the `template.py`
script is provided as a blueprint.

This component communicates through the following HTTP requests:

* Data messages:

    - GET `/algn`

* Control messages:

    - PATCH `/norms` changes the normative system
    - PATCH `/path_length` changes the length of the paths used to sample
      outcomes
    - PATH `/path_sample` changes the number of paths sampled to compute the
      alignment

# Deployment

Clone this repository and develop your model and value semantics functions
following the blueprint in `template.py`:

```bash
$ git clone https://github.com/VALAWAI/C2_alignment_calculator.git
```

Build your Docker image in your directory of the component repository:

```bash
$ cd /path/to/c2_alignment_calculator
$ docker build -t c2_alignment_calculator .
```

Run a Docker container with your C2 Alignment Calculator:

```bash
$ docker run --rm -d \
  --network valawai \
  --name c2_alignment_calculator \
  --mount type=bind,src="$(pwd)",target=/app \
  -p 5432:5000 \
  -e MODEL=my_model \
  c2_alignment_calculator
```

The environment variable `MODEL` refers to the script where you have defined
your model (do not include the .py extension).

Once the container is up and running, use `curl` to communicate with the
component:

```bash
$ curl http://localhost:5432/algn
{
  "algn": 0.9048607793661254
}
```

```bash
$ curl -X PATCH http://localhost:5432/path_sample -d 200
{}
```
