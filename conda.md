# Anaconda cheatsheet

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
```

### Switching between Envs (`anaconda` folder must be the first in PATH)
```
source activate ENVNAME
```
