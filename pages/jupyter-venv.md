## Using Jupyter within a virtual environment on macOS

This short guide will show you how to use Jupyter notebooks from within a Python virtual environment.

Firstly, check the list of available kernels: `jupyter kernelspec list`.  You should see one kernel e.g. `python3` which is your system installation of Python.

Then, create a virtual environment in your chosen way and install Jupyter.  For example:
```bash
cd ~/environments
python3 -m venv test_env
source test_env/bin/activate
pip install jupyter
```

Then, go to `~/Library/Jupyter/kernels/` and create a new directory e.g. `my_new_kernel`.

Within this directory create a new file called `kernel.json`.

Insert the following into this file:

```json
{
 "argv": [
  "/Users/imran/environments/test_env/bin/python",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "my_new_kernel",
 "language": "python"
}
```

There are two things which you should edit:
* The first element of the `argv` list should be the absolute path to the Python executable within your virtual environment.
* You can edit `display_name` to whatever you choose.  This will come up as an option under 'New' when creating a new notebook.

If you run `jupyter kernelspec list` again you should see your newly created kernel.

More information about how to configure Jupyter kernels can be found [here](https://jupyter-client.readthedocs.io/en/stable/kernels.html#kernel-specs).

---
[Home](../index.md)
