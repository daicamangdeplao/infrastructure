# Quick Notes

If  all your projects had a Dockerfile and docker-compose.yml then "new developer onboarding" would be:
* git clone
* docker-compose up


network:


## Components

* postgresql
* dbeaver: docker run --name cloudbeaver -d --rm -ti -p 8080:8978 -v /opt/cloudbeaver/workspace dbeaver/cloudbeaver:latest

## References

* https://www.baeldung.com/ops/postgresql-docker-setup
* https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/
* https://hub.docker.com/_/postgres