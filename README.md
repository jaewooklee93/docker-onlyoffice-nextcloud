## OnlyOffice Server and Nextcloud Docker installation
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

4. 약 2분간 기다려서 `http://localhost` 로 접속 가능한지 확인한다.

5. Go to the project folder and run the `set_configuration.sh` script:
    ```bash
    bash set_configuration.sh
    ```
**Please note**: The default JWT (secret key) is enabled in ONLYOFFICE Document Server. It is recommended to specify your own secret key in the Nextcloud administrativ

6. 우상단 프로필 아이콘 > Administration settings > ONLYOFFICE > Common settings > Checkbox 모두 선택 > Save

Now you can enter Nextcloud and create a new document. It will be opened in ONLYOFFICE Document Server. 

## Known Issues
- 도메인 및 SSL/TLS 관련하여 Cloudflare Proxy나 flexible SSL 적용된 경우 성능이 매우 저하되고 Office 기능은 아예 작동하지 않음: [nextcloud/all-in-one](https://github.com/nextcloud/all-in-one?tab=readme-ov-file#notes-on-cloudflare-proxytunnel)
- Cloudflared Tunnel 사용시에도 소규모 이용은 가능하나 localhost에 비해 10배 정도 느림
- 인트라넷에서 IP 주소로 접근시에는 성능 저하 없을듯

## 외부 도메인 연결하기 (cloudflare tunnel 기준)
- `your.tunnel.domain` --> `localhost:80`
- config/config.php 수정 필요

    ```bash
    # 꺼내기
    $ docker compose cp app:/var/www/html/config/config.php .
    $ vi ./config.php
    ```
    
config.php 파일에 접속할 주소나 아이피를 추가하여 아래와 같이 같이 수정한다

    ```php
        'trusted_domains' =>
            array (
                0 => 'localhost',
                1 => 'nginx-server',
                2 => 'your.tunnel.domain'  // 필요시 추가
                3 => '111.222.333.444'     // 필요시 추가
            ),
    ```

    ```bash
    # 도로 집어넣기
    $ docker compose cp ./config.php app:/var/www/html/config/config.php
    $ docker compose exec app chown 33:33 /var/www/html/config/config.php
    ```
