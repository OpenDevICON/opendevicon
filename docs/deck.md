# Deck

## Getting juypter notebook

[**Jupyter Notebook**](https://jupyter.org/) is an application widely used in Data Science domain for creating and sharing documents that contains Live code, Equations, Visualizations, and Explanatory text.

{% hint style="info" %}
While Jupyter runs code in many programming languages, **Python** is a requirement(Python 3.3 or greater, or Python 2.7) for installing Jupyter Noteboook.
{% endhint %}

### Installing Jupyter Notebook using Conda

### conda
We recommend installing Python and Jupyter using the conda package manager. The [miniconda](https://docs.conda.io/en/latest/miniconda.html) distribution includes a minimal Python and conda installation.

Then you can install the notebook with:
```bash
$ conda install -c conda-forge notebook
```

### pip
If you use **pip**, you can install it with:
```bash
$ pip install notebook
```

To run the notebook, run the following command 
```bash
$ jupyter-notebook
```

## Installing PythonSDK
The source code is found on [GitHub](https://github.com/icon-project/icon-sdk-python)

ICON SDK for Python development and execution requires the following environments.

- Python
  - Version: Python 3.6+
  - IDE: Pycharm is recommended.

It can be installed using pip as follows:
  ```bash
  $ pip install iconsdk
  ```
