# tix-deploy

Script and instructions to run TiX system in a docker compose environment.

## Secrets

The different services require access secrets. To avoid hardcoding them in the code (and leaving them leaked in the repository), the Docker compose file will load them from an [environment file](https://docs.docker.com/compose/environment-variables/). The easiest way is to create a file named `.env` in the same directory the `docker-compose.yml` file is located with the following content (replacing with real values):

```shell
# rabbitMQ credentials
RABBITMQ_USERNAME=<name>
RABBITMQ_PASSWORD=<pass>

# API admin user credentials
TIX_API_ADMIN_USERNAME=<user>
TIX_API_ADMIN_PASSWORD=<pass>
RECAPTCHA_SECRET_KEY=<secret-key>

# port in the host where the frontend is served. For local deployments
# you can use another port (like 3000)
WEB_PORT=80

# API location. When deploying with compose there's no need to change them
# because they are used for communicating services inside Docker's network
TIX_API_HOST=tix-api
TIX_API_PORT=3001
TIX_API_HTTPS=False
```

## Execute

Once the `.env` file is properly set up, just run:

```shell
docker-compose up
```

### Restart deploy

The compose will create some persistent volumes. These volumes are persisted even if you destroy the containers by executing `docker-compose down`. If you want to run the deployment from scratch, you'll have to delete them. You can list the existing volumes by using the `docker volume ls` command and then delete by using `docker volume rm <volume name>`.
