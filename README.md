## Document Server and Nextcloud Docker installation
- NextCloud: 오픈소스 Dropbox
- OnlyOffice: 오픈소스 Google Docs/Spreadsheet

## Installation

1. Get the latest version of this repository running the command:

    ```
    git clone https://github.com/ONLYOFFICE/docker-onlyoffice-nextcloud # Original
    git clone https://github.com/jaewooklee93/docker-onlyoffice-nextcloud # Modified docker-compose.yml + www.conf
    cd docker-onlyoffice-nextcloud
    ```

* `/usr/local/etc/php-fpm.d/www.conf` 파일에서 php-fpm 동시 접속자 수를 늘려주어야 한다
    (Ref. https://docs.nextcloud.com/server/21/admin_manual/installation/server_tuning.html#tune-php-fpm )

```bash
docker create --name temp_container nextcloud:fpm
docker cp temp_container:/usr/local/etc/php-fpm.d/www.conf .
docker rm temp_container
sed -i 's/^pm\.max_children\s*=.*/pm.max_children = 120/' www.conf
sed -i 's/^pm\.start_servers\s*=.*/pm.start_servers = 12/' www.conf
sed -i 's/^pm\.min_spare_servers\s*=.*/pm.min_spare_servers = 6/' www.conf
sed -i 's/^pm\.max_spare_servers\s*=.*/pm.max_spare_servers = 18/' www.conf
```
본 repo에는 미리 생성해둠

2. Run Docker Compose:

    ```
    docker compose up -d
    ```

3. Now launch the browser and enter the webserver address. The Nextcloud wizard webpage will be opened. Enter all the necessary data to complete the wizard.
기본값을 넣어두어서 이부분은 자동으로 진행됨.

4. Go to the project folder and run the `set_configuration.sh` script:

    ```
    bash set_configuration.sh
    ```

**Please note**: The default JWT (secret key) is enabled in ONLYOFFICE Document Server. It is recommended to specify your own secret key in the Nextcloud administrative configuration and [ONLYOFFICE Docs](https://helpcenter.onlyoffice.com/installation/docs-configure-jwt.aspx).

Now you can enter Nextcloud and create a new document. It will be opened in ONLYOFFICE Document Server. 

- http://localhost로 접속하면 됨.

