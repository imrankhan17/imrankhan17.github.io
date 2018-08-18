## Opening a Jupyter Notebook on an EC2 instance

Install Python3 and create a virtual environment:  
 - `yes | sudo yum install python36`
 - `python3 -m venv env`
 - `source env/bin/activate`

Install jupyter:
 - `pip install jupyter`

In a new terminal:
 - `ssh -i <key pair location> -L 8000:localhost:8888 ec2-user@<IPv4 Public IP>` 
 - e.g. `ssh -i ~/my_key.pem -L 8000:localhost:8888 ec2-user@51.120.129.202`

In the previous terminal:
 - `jupyter notebook`

In a browser:
 - `localhost:8000`

Enter the token shown in the terminal.  

---
[Home](../index.md)
