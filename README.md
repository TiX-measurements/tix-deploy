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

# this is the secret used by Google's ReCAPTCHA service. If you change this key you'll also
# have to change the corresponding client key `tix-web` and `tix-app`
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

### IP to AS

The [service `tix-iptoas`](https://github.com/TiX-measurements/ip_to_as) will initialize the MySQL database with the mappings from the IP prefixes to autonomous system organization names (used for when the application creates reports). This service is manually executed because it takes a while to run. To run it just execute:

```shell
# override the default "dummy" command used in the compose file by setting the entrypoint
# explicitly
docker compose run --entrypoint 'sh /app/exe.sh' tix-iptoas
```

### Restart deploy

The compose will create some persistent volumes. These volumes are persisted even if you destroy the containers by executing `docker-compose down`. If you want to run the deployment from scratch, you'll have to delete them. You can list the existing volumes by using the `docker volume ls` command and then delete by using `docker volume rm <volume name>`.

## Updating services

These are the steps used to update a service in production without downtime:

 1. Push the new service image (with tag `latest`) to the Docker's registry (for example `docker push tixmeasurements/tix-api:latest`)
 2. Log into the server where `tix-deploy` is cloned
 3. Download the new image: `docker compose pull`
 4. Restart the updated image: `docker compose up --force-recreate -d tix-api` (or whatever service you want to update)
