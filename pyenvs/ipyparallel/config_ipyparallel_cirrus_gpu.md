Instructions for setting up an ipyparallel configuration on Cirrus (GPU)
========================================================================

The instructions below start with the loading of the `miniconda3/4.9.2-gpu` module
within the user's account on the Cirrus login node.


Setup initial environment
-------------------------

```bash
module load miniconda3/4.9.2-gpu

ipython profile create --parallel --profile=mpi
```


Edit the `~/.ipython/profile_mpi/ipcluster_config.py` file
----------------------------------------------------------

Add the following line.

```bash
c.IPClusterEngines.engine_launcher_class = 'MPIEngineSetLauncher'
```


Edit the `~/.ipython/profile_mpi/ipengine_config.py` file
----------------------------------------------------------

Add the following lines.

```bash
c.EngineFactory.use_mpi = True
c.MPI.use = 'mpi4py'
```


Submit your Jupyter password
----------------------------

Start a python session from within your Cirrus login account and
enter your Jupyter password. In the example below the password
is `pyjuptr`.

```bash
In [1]: from notebook.auth import passwd
In [2]: passwd(algorithm='sha1')
Enter password: pyjuptr
Verify password: pyjuptr
Out[2]: 'sha1:779e3731165d:674576a3aa1446d6e63dbed75d0f90ec3b2a1da7'
exit()
```


Create the Jupyter configuration file
-------------------------------------

Return to your Cirrus login account and run the following command.

```bash
jupyter notebook --generate-config
```


Edit the `~/.jupyter/jupyter_notebook_config.py` file
-----------------------------------------------------

Add the following lines (the hash must match that generated for your Jupyter password).

```bash
c.NotebookApp.allow_password_change = False
c.NotebookApp.password = 'sha1:779e3731165d:674576a3aa1446d6e63dbed75d0f90ec3b2a1da7'
```

FYI, the ipyparallel package is supported by a [source code repository](https://github.com/ipython/ipyparallel) and [extensive documentation](https://ipyparallel.readthedocs.io/en/latest/).