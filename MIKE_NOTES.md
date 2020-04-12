# Notes

My notes on testing using Stefan's scripts.

## Builder

Already done by Stefan. Only two additions:

- On Windows try `docker run --detach --hostname gitlab.example.com --publish 443:443 --publish 80:80 
--publish 22:22 --name gitlab-11.2 --restart always --volume glconfig:/etc/gitlab --volume gllogs:/var/log/gitlab --volume gldata:/var/opt/gitlab gitlab/gitlab-ce:11.2.0-ce.0`.
- You need to wait 5-10 minutes for the docker image to boot.

## Runner

## Mocker