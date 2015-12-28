# Anaconda cheatsheet

### Remove Anaconda
```
rm -rf ~/anaconda
```

### Version
```
conda --version
```

### Update
```
conda update conda
```

### List
```
conda list
```

### Envs
```
conda info --envs
```

### Create Env
```
conda create --name bunnies python=3 astroid babel
conda create --name snowflakes biopython
conda create --name flowers --clone snowflakes
```

### Switching between Envs
`anaconda` folder must be the first in PATH
```
source activate ENVNAME
```

### Remove Env
```
conda remove --name flowers --all
```

### Python Version
```
python --version
```

### Search Package
```
conda search --full-name python
```
search any package contains the `python` in name
```
conda search python
```

### Install Package Within Env
```
conda install twisted
```

For packages not in conda: anaconda
```
conda install --channel https://conda.anaconda.org/pandas bottleneck
```

For packages not in conda: pip
```
pip install see
```

### Remove Package Within Env
```
conda remove twisted
```

For pip-installed packages
```
pip uninstall see
```

### Help
```
conda create --help
conda remove --help
```
