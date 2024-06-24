## OnlyOffice Document Server and Nextcloud Docker installation
- NextCloud: 오픈소스 Dropbox
- OnlyOffice: 오픈소스 Google Docs/Spreadsheet

## Installation

1. Get the latest version of this repository running the command:

```bash
# Original repo
git clone https://github.com/ONLYOFFICE/docker-onlyoffice-nextcloud

# Modified docker-compose.yml + www.conf (This repo)
git clone https://github.com/jaewooklee93/docker-onlyoffice-nextcloud

cd docker-onlyoffice-nextcloud
```

2. `www.conf` 설정파일 생성

- `app` 컨테이너의 `/usr/local/etc/php-fpm.d/www.conf` 파일에서 php-fpm 동시작업 수를 늘려주어야 함: [docs.nextcloud.com](https://docs.nextcloud.com/server/21/admin_manual/installation/server_tuning.html#tune-php-fpm)

```bash
docker create --name temp_container nextcloud:fpm
docker cp temp_container:/usr/local/etc/php-fpm.d/www.conf .
docker rm temp_container
sed -i 's/^pm\.max_children\s*=.*/pm.max_children = 120/' www.conf
sed -i 's/^pm\.start_servers\s*=.*/pm.start_servers = 12/' www.conf
sed -i 's/^pm\.min_spare_servers\s*=.*/pm.min_spare_servers = 6/' www.conf
sed -i 's/^pm\.max_spare_servers\s*=.*/pm.max_spare_servers = 18/' www.conf
# ./www.conf 파일이 생성됨
```

본 repo에는 미리 생성해 두었음.

3. (docker-compose.yml에서 NextCloud Admin ID/PW만 설정한 후) Run Docker Compose: 

    ```bash
    docker compose up -d
    ```

3. Go to the project folder and run the `set_configuration.sh` script:
    ```bash
    bash set_configuration.sh
    ```
**Please note**: The default JWT (secret key) is enabled in ONLYOFFICE Document Server. It is recommended to specify your own secret key in the Nextcloud administrative configuration and [ONLYOFFICE Docs](https://helpcenter.onlyoffice.com/installation/docs-configure-jwt.aspx).

4. `http://localhost` 로 접속하면 됨.

Now you can enter Nextcloud and create a new document. It will be opened in ONLYOFFICE Document Server. 

## Known Issues
- 도메인 및 SSL/TLS 관련하여 Cloudflare Proxy나 flexible SSL 적용된 경우 성능이 매우 저하되고 Office 기능은 아예 작동하지 않음: [nextcloud/all-in-one](https://github.com/nextcloud/all-in-one?tab=readme-ov-file#notes-on-cloudflare-proxytunnel)
- Cloudflared Tunnel 사용시 소규모 이용에는 문제가 없으나 대규모 이용시 어떨지는 모른다.
    - `<tunnel-domain>` --> `localhost:80`
- 인트라넷에서 IP 주소로 접근시에는 문제 없을 것
