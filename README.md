# Local Nextcloud instance with dabase backup

Get a copy of the `data` folder, but exclude user and app data.
Go to `/srv/nextcloud_jdav` and run the following command to save a compressed tar archive in your home directory.

`sudo tar -czf ~/nextcloud.tar --exclude='data/nextcloud/data' ./data`

Fetch the file from YOUR machine with SCP

- `scp <user>@vps2.jdav-freiburg.de:~/nextcloud.tar <target-dir>`
- `sudo tar --same-owner -xf nextcloud.tar <target-dir>`

Create the missing `data` folder, create the mandatory `.ncdata` file and give the correct permissions

- `mkdir ./data/nextcloud/data`
- `echo '# Nextcloud data directory' | sudo tee ./data/nextcloud/data/.ncdata`
- `chown -R www-data:www-data ./data/nextcloud/data`

Make sure to run `chown -R www-data:www-data ./data/nextcloud/html`

TEMPORARILY open Port 80 in `docker-compose.yaml` for the service `web`

Run `docker-compose up -d`.

Disable the "Nextcloud Office" app, because we didn't copy it's huge data-folder.
If not disabled, it will lead to `500 Internal Server Error`

`docker compose exec -it --user www-data app /var/www/html/occ app:disable richdocuments`

Go to [http://localhost/](http://localhost/).

In case you have trouble with https being enforced, go to `./data/nextcloud/html/config/config.php` and set `'overwriteprotocol' => 'http',`.
