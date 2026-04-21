# Gitea restore (rootless)

```
docker cp gitea-dump.zip CONTAINER_ID:/tmp

# open bash session in container
docker exec --user git -it CONTAINER_ID bash

# unzip your backup file within the container
unzip gitea-dump.zip

# restore the app.ini
mv app.ini /etc/gitea/app.ini

# restore the gitea data
rm -rf /var/lib/gitea/custom
rm -rf /var/lib/gitea/git
mv data/* /var/lib/gitea

# restore the repositories itself
mkdir /var/lib/gitea/git/repositories
mv repos/* /var/lib/gitea/git/repositories

# adjust file permissions
chown -R git:git /etc/gitea/app.ini /var/lib/gitea

# Regenerate Git Hooks
/usr/local/bin/gitea -c '/etc/gitea/app.ini' admin regenerate hooks
```

and if the ROOT_URL changed, replace it with new one in /var/lib/docker/volumes/gitea_gitea-config/_data/app.ini

command for fixing runner with custom config:
```
act_runner -c config.yaml daemon
```
