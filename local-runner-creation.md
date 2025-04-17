# Creating a locally runnable GitLab runner using Docker

> [!CAUTION]
> This is not a best practice, and there are likyle numberous things we want to do differently when creating the actual runners

## Fetch the image and create a volume
These need to be done on the first run only:
```
docker pull gitlab/gitlab-runner
docker volume create gitlab-runner-demo-config
```

## Registering the runner
You need to create a runner in the repo settings (this repo assumes it has tag kp-demo), and then provide its token and url. This only needs to be done once per runner (because the config is saved in the previously-created volume).
```
docker run -v gitlab-runner-demo-config:/etc/gitlab-runner gitlab/gitlab-runner register --url https://gitlab.ci.csc.fi --token [kopioi gitlabin kälistä kun luot uutta runneria] --executor shell --non-interactive
```

This also starts the runner.

## Restarting the runner (e.g. after reboot)
If there's a pre-existing container with the same name (see `docker container ls`) you need to delete it (`docker container remove`). After that, you can
```
docker run --name gitlab-runner-demo --restart always -v gitlab-runner-demo-config:/etc/gitlab-runner gitlab/gitlab-runner
```
