## Creating a database instance in Docker

Pull a [MariaDB](https://hub.docker.com/_/mariadb/) Docker image:
```bash
docker pull mariadb
```

Start a container and specify a root password:
```bash
docker run -d --name my-db -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 mariadb
```

Connect to MariaDB console inside container:
```bash
docker exec -it my-db mysql -p
```

---
[Home](../index.md)
