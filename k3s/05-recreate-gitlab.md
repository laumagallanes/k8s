# Recreate GitLab After a Failure

This guide describes how to rebuild a containerized GitLab instance from its Docker volumes.

## Objective

Recover GitLab on another machine or after a major failure without depending on the old container filesystem.

The key idea is straightforward:

- preserve the data volumes
- recreate empty volumes on the target machine
- restore the data into them
- start GitLab again with the same volume layout

## Volumes to preserve

The original deployment persisted at least these volumes:

- `gitlab-config`
- `gitlab-logs`
- `gitlab-data`

These volumes contain the real state of the instance.

## Step 1: stop GitLab on the source host

For better consistency, stop the container before taking the backup:

```bash
docker stop gitlab
```

## Step 2: back up each volume

Use a temporary container to archive the content of each volume.

Example for `gitlab-config`:

```bash
docker run --rm \
  -v gitlab-config:/data:ro \
  -v $(pwd):/backup \
  ubuntu \
  tar cvf /backup/gitlab-config.tar /data
```

Repeat the same process for:

- `gitlab-logs`
- `gitlab-data`

## Step 3: copy the archives to the target host

Transfer the `.tar` files using the method that fits your environment, such as:

- `scp`
- `rsync`
- secure file transfer through your usual admin workflow

## Step 4: recreate empty volumes on the destination host

```bash
docker volume create gitlab-config
docker volume create gitlab-logs
docker volume create gitlab-data
```

## Step 5: restore each archive into the new volumes

Example for `gitlab-config`:

```bash
docker run --rm \
  -v gitlab-config:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar xvf /backup/gitlab-config.tar -C /data --strip 1
```

Repeat for the remaining volumes.

Note: the `--strip 1` flag may need adjustment depending on how the tarball was created.

## Step 6: start GitLab again

Once the volumes are restored, run GitLab with the same layout.

Example pattern:

```bash
docker run -d --name gitlab --restart always -p 80:80 \
  -v gitlab-config:/etc/gitlab:Z \
  -v gitlab-logs:/var/log/gitlab:Z \
  -v gitlab-data:/var/opt/gitlab:Z \
  gitlab/gitlab-ce:latest
```

## SRE notes

### Consistency first

If possible, stop GitLab before backing up. Hot copies may work, but they are a bigger gamble.

### Watch the image version

Do not casually restore old data into an incompatible GitLab version. Pin or verify the version used on the target host.

### Data matters more than the container

You do not need to preserve the old container image itself if you can start a compatible GitLab image and restore the volumes correctly.

### Check networking assumptions

If the new host changes:

- hostname
- reverse proxy path
- DNS behavior
- external port mapping

then GitLab may start but still behave badly for users or integrations.

## Quick validation after restore

After recovery, verify:

- the web UI loads
- repositories still exist
- users can authenticate
- CI pipelines can connect
- the instance behaves normally after restart
