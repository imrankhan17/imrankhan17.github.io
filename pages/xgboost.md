## Installing xgboost on an EC2 Linux instance

Install Python3 and create a virtual environment:  
 - `yes | sudo yum install python36`
 - `python3 -m venv env`
 - `source env/bin/activate`

Add the virtual environment to the python path:
 - `export PYTHONPATH="/home/ec2-user/.local/lib/python3.6/site-packages/"`

Then install git and gcc:  
 - `sudo yum install git`
 - `sudo yum install gcc-c++`

Installing xgboost from source:  
 - `git clone --recursive https://github.com/dmlc/xgboost.git`
 - `cd xgboost`
 - `make`
 - `cd python-package`
 - `python setup.py install --user`

 Inside a python REPL:  
 - `import xgboost as xgb`
