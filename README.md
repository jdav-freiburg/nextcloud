# JDAV Nextcloud

## Local setup (development only)

This describes how to create a local Nextcloud instance with a database backup.
Only necessary for local development, testing, etc.

### Get a copy of all necessary data

Get a copy of the `data` folder, but exclude user data and the large `preview` sub-folder.
This way the `data` folder should be around 4-5GB and the compressed version (`-z` option) should be around 1.3GB.
Go to `/srv/nextcloud_jdav` and run the following command to save a compressed tar archive in your home directory.

```shell
ssh <user>@vps2.jdav-freiburg.de
```

If the file `/srv/nextcloud.tar.gz` does not already exist, create it with the following commands

```shell
# Create new archive (-c) with file name (-f)
# Append necessary folder `data` to existing archive (-r) with file name (-f)
# and exclude the content of the `data` sub-folder,
# which mostly contains data uploaded by users into their user folders,
# except for the necessary items `.ncdata` and `appdata_ocadc83b19e7`,
# which we will add in the following steps
sudo tar -cf /srv/nextcloud.tar /srv/nextcloud_jdav/data --exclude='data/nextcloud/data'

# Append to existing archive (-r) with file name (-f)
# The file `.ncdata` is required by Nextcloud to mark this as the Nextcloud data folder
sudo tar -rf /srv/nextcloud.tar /srv/nextcloud_jdav/data/nextcloud/data/.ncdata

# Append to existing archive (-r) with file name (-f)
# and exclude the `preview` sub-folder (>= 34GB)
# The folder `appdata_ocadc83b19e7` is required to be able to access all functions in the frontend
sudo tar -rf /srv/nextcloud.tar /srv/nextcloud_jdav/data/nextcloud/data/appdata_ocadc83b19e7 \
    --exclude='/srv/nextcloud_jdav/data/nextcloud/data/appdata_ocadc83b19e7/preview'

# Gzip converts it to `/srv/nextcloud.tar.gz`
# It reduces the file size from ~4.4GB to ~1.3GB for faster transfer
gzip /srv/nextcloud.tar
```

Fetch the file from YOUR machine with SCP

```shell
scp <user>@vps2.jdav-freiburg.de:/srv/nextcloud.tar.gz <target-dir>
```

Extract the archive while keeping all original permissions

```shell
sudo tar --same-owner -xzf nextcloud.tar.gz --directory=<target-dir/with/your/docker-compose.yaml>
```

It still might be necessary to set the correct permissions for Nextclouds `data` folder

```shell
chown -R www-data:www-data <you-folder>/data/nextcloud/data
```

### Start and configure the local instance

Deactivate forced use of https

```shell
sudo sed -i "s/'overwriteprotocol' => 'https'/'overwriteprotocol' => 'http'/g" <you-folder>/data/nextcloud/html/config/config.php
```

Start the local instance

```shell
docker compose -f docker-compose.yaml -f docker-compose.dev.yaml up -d
```

You can access the local Nextcloud instance at [http://localhost/](http://localhost/).

> [!INFO]
> **Must** be **HTTP** and not https!

### Working with the local instance

A few helpful infos and commands to get you started.

#### Nextcloud command-line interface OCC

Nextcloud provides the command-line interface (CLI) [OCC](https://docs.nextcloud.com/server/latest/admin_manual/occ_command.html) for administrative tasks.

How to run the base command `occ` in the container

```shell
docker compose exec -ti --user www-data app /var/www/html/occ
```

#### Troubleshooting

Error logs: [Admin > Logging](http://localhost/settings/admin/logging)

Try disabling the following apps if you runinto `500 Internal Server Error` in the frontend

```shell
# Disable the Calendar
# It would prevent the frontend from loading the `Apps` page
docker compose exec -ti --user www-data app /var/www/html/occ app:disable calendar

# We did not copy the huge data folder of the "Nextcloud Office" app
# It would prevent the frontend from loading after you logged in
docker compose exec -it --user www-data app /var/www/html/occ app:disable richdocuments
```
