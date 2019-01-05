## Connecting multiple containers in Docker

In this tutorial we will use Docker to create separate Python and database containers and place them in their own network such that they are able to communicate with each other.  In Docker this can be done manually using normal `docker` commands or we can use a tool called Docker Compose.

The final code from this tutorial is on GitHub [here](https://github.com/imrankhan17/docker-compose-example).

### Manual method

Firstly, let's view what Docker networks we have currently:
```bash
docker network ls
```

Create a new network which we will call `appnet`:
```bash
docker network create appnet
```

Now onto the containers.  We will use MariaDB for our database container:
```bash
docker run --rm -d --name=db --network=appnet -e MYSQL_ROOT_PASSWORD=password mariadb
```

Start a temporary Python container within the same network (specified by the `--network` flag) and open up a shell prompt:
```bash
docker run --rm -it --name=app --network=appnet python:alpine sh
```

Install PyMySQL with `pip install PyMySQL`.

Now within a `python` prompt try:
```python
import pymysql.cursors
connection = pymysql.connect(host='db', user='root', password='password')
```

The database host is the name of the MariaDB container that we gave it.  If this runs without errors, then we have managed to get our Python container to talk to our database container.

Let's now create a more permanent container.  Exit out of the Python container.  Insert the Python code above into a file called `app.py`.  Create another file called `Dockerfile` and insert into it the following:
```dockerfile
FROM python:alpine
COPY . .
RUN pip install pymysql
CMD ["python", "app.py"]
```

Build the image:
```bash
docker build -t pythonapp .
```

Create a container (make sure the database container from above is still running):
```bash
docker run --rm --name=app --network=appnet pythonapp
```

If this returns nothing, then it should have worked.  Add a `print('It worked!')` to the end of `app.py` to check (remember to rebuild the image first).

### Using docker-compose

Whilst it's only really a few Docker commands to create this very simple network of containers, it can get unwieldy if we had many more containers and multiple networks.  Instead we can use Docker Compose which uses a YAML file to define all our containers and network configuration.

We will need to create a file called `docker-compose.yaml` in the root directory of our project.

Let's insert the following into this file:
```yaml
version: "3"
services:
  db:
    image: mariadb:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
  python:
    build: .
```

You can see that we have two "services" - these are our containers i.e. the database and the Python script.  You can also see that the file conveys pretty much the same information as our Docker commands from before.  More information about the Compose file format can be found [here](https://docs.docker.com/compose/compose-file/).  Note that we do not need to specify anything about a network since Docker Compose will automatically assign one for all the services.

We will also need to tweak the Python script to account for the MariaDB instance taking a bit of time to start up.  You can control the order in which services start up, but the [docs](https://docs.docker.com/compose/startup-order/) state that:
> Compose does not wait until a container is “ready” ... - only until it’s running.

In our case this means that the MariaDB instance might be pingable at the start, but it doesn't mean we can run queries on it yet.

Let's put this in `app.py` instead:
```python
import pymysql.cursors
import time

while True:
    try:
        connection = pymysql.connect(host='db', user='root', password='password')
        print('WE ARE IN!', flush=True)
        break
    except:
        print('NOT YET IN.', flush=True)
        time.sleep(1)
```

The container will now try to connect to the database every one second until it works.

Finally, we can start up our network of containers with simply:
```bash
docker-compose up
```

This will build the images and start each container.  You should see a load of log messages mostly coming from the database container but also a few `NOT YET IT.` logs from the Python container.  At the end you should see the final `WE ARE IN!` log message confirming that it worked.

`docker-compose ps` will show a list of containers that have been created through Compose.  `docker-compose down` will stop and remove all the containers.

We have now laid the groundwork to create a proper Python application that utilises a database and also make sure that it's all within it's own network.

---
[Home](../index.md)
