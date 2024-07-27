- pull the docker postgres image instance

```javascript

docker pull postgres


```

- run the docker postgres image instance

```javascript

docker run --name docker-postgres -e POSTGRES_PASSWORD=password -e POSTGRES_USER=postgres -e POSTGRES_DB=mydatabase -d -p 5433:5432 postgres

```
