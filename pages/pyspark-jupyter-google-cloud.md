## Running PySpark in a Jupyter Notebook on Google Cloud

Create a cluster using the Google Cloud [Dataproc](https://cloud.google.com/dataproc/) console or the command line:  

```bash
gcloud dataproc \
    --region europe-west1 \
    clusters create {cluster name} \
    --subnet default \
    --zone "" \
    --master-machine-type n1-standard-8 \
    --master-boot-disk-size 500 \
    --num-workers 5 \
    --worker-machine-type n1-standard-16 \
    --worker-boot-disk-size 500 \
    --image-version 1.2 \
    --project {project name}
```

Use port forwarding to login to the instance:
 - `ssh -i <key pair location> -L 8000:localhost:8888 <username>@<IPv4 Public IP>` 
 - e.g. `ssh -i ~/.ssh/google_compute_engine -L 8000:localhost:8888 imran@31.210.59.152`

Install pip and other packages required on Debian:
 - `yes | sudo apt install python-pip build-essential python-dev`
 - `sudo python -m pip install jupyter`

Create new environment variables:
 - `export PYSPARK_DRIVER_PYTHON=jupyter`
 - `export PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port=8888'`

Start a Jupyter session:
 - `pyspark`

In a browser:
 - `localhost:8000`

Enter the token shown in the terminal.  

---
[Home](../index.md)
