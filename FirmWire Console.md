# IPython/Jupyter console

Execute `firmwire.py` with `--console` flag:

```
python3 firmwire.py modem.bin --console
```

Connect to the container using new terminal, see [[FirmWire Setup#Connect to existing container]]

Connect to the IPython kernel:
```
jupyter-console --existing
```

```
root@a127cb9da190:/firmwire# jupyter-console --existing
Jupyter console 6.4.3

Python 3.8.10 (default, Feb  4 2025, 15:02:54) 
Type 'copyright', 'credits' or 'license' for more information
IPython 7.13.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]:             
```

TODO: Find a way to expose this to host