<br/>
<div align="center">
<p align="center">
  <img width="234" src="docs/media/logo-full.png"/>
</p>
  <p>
     <a href="https://github.com/netbirdio/netbird/blob/main/LICENSE">
       <img src="https://img.shields.io/badge/license-BSD--3-blue" />
     </a> 
   <a href="https://www.codacy.com/gh/netbirdio/netbird/dashboard?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=netbirdio/netbird&amp;utm_campaign=Badge_Grade"><img src="https://app.codacy.com/project/badge/Grade/e3013d046aec44cdb7462c8673b00976"/></a>
    <br>
    <a href="https://join.slack.com/t/netbirdio/shared_invite/zt-vrahf41g-ik1v7fV8du6t0RwxSrJ96A">
        <img src="https://img.shields.io/badge/slack-@netbird-red.svg?logo=slack"/>
     </a>    
  </p>
</div>


<p align="center">
<strong>
  Start using NetBird at <a href="https://netbird.io/pricing">netbird.io</a>
  <br/>
  See <a href="https://netbird.io/docs/">Documentation</a>
  <br/>
   Join our <a href="https://join.slack.com/t/netbirdio/shared_invite/zt-vrahf41g-ik1v7fV8du6t0RwxSrJ96A">Slack channel</a>
  <br/>
</strong>
</p>
<br>

### Hướng dẫn cài đặt Netbird Server Docker
- Tài liệu: [Netbird](https://docs.netbird.io/selfhosted/selfhosted-guide)

***Yêu cầu***
- [Docker Compose](https://docs.docker.com/compose/install/)
- jq
  
**Bước 1: Tải code mới nhất**

```#!/bin/bash
REPO="https://github.com/netbirdio/netbird/"
# this command will fetch the latest release e.g. v0.8.7
LATEST_TAG=$(basename $(curl -fs -o/dev/null -w %{redirect_url} ${REPO}releases/latest))
echo $LATEST_TAG

# this command will clone the latest tag
git clone --depth 1 --branch $LATEST_TAG $REPO
```
- Đến thư mục build
`cd netbird/infrastructure_files/`

**Bước 2: Chỉnh sửa cấu hình**

- Tạo file `setup.env` trong đường dẫn `netbird/infrastructure_files/` với nội dung
  
```
NETBIRD_DOMAIN="192.168.15.6"
NETBIRD_AUTH_OIDC_CONFIGURATION_ENDPOINT="https://id.lab.linksafe.vn/realms/netdev/.well-known/openid-configuration"
NETBIRD_USE_AUTH0=false
NETBIRD_AUTH_CLIENT_ID="netbird-client"
NETBIRD_AUTH_SUPPORTED_SCOPES="openid profile email offline_access api"
NETBIRD_AUTH_AUDIENCE="netbird-client"

NETBIRD_DISABLE_LETSENCRYPT=true

NETBIRD_AUTH_DEVICE_AUTH_CLIENT_ID="netbird-client"

NETBIRD_MGMT_IDP="keycloak"
NETBIRD_IDP_MGMT_CLIENT_ID="netbird-backend"
NETBIRD_IDP_MGMT_CLIENT_SECRET="CxpAdSREUwdYIRlDlwYwxCBeaV7rL86K"
NETBIRD_IDP_MGMT_EXTRA_ADMIN_ENDPOINT="https://id.lab.linksafe.vn/admin/realms/netdev"

```

- Với các thông số cần chỉnh sửa như sau:
  
\- NETBIRD_DOMAIN: IP của máy đang build  
\- NETBIRD_AUTH_OIDC_CONFIGURATION_ENDPOINT: https://id.lab.linksafe.vn/realms/{realms}/.well-known/openid-configuration. Ví dụ realms ở đây là `netdev`  
\- NETBIRD_AUTH_CLIENT_ID, NETBIRD_AUTH_AUDIENCE, NETBIRD_AUTH_DEVICE_AUTH_CLIENT_ID: `netbird-client` sẽ là tên client được tạo trong Keycloak  
\- NETBIRD_IDP_MGMT_CLIENT_ID: `netbird-backend` sẽ là tên client được tạo trong Keycloak  
\- NETBIRD_IDP_MGMT_CLIENT_SECRET: là client secret của `netbird-backend`  
\- NETBIRD_IDP_MGMT_EXTRA_ADMIN_ENDPOINT: https://id.lab.linksafe.vn/admin/realms/{realms} tương tự sẽ thay bằng realms của Keycloak

**Bước 3: Cấu hình IDP Keycloak**

  **Tạo realms**
- Truy cập `https://id.lab.linksafe.vn/` và đăng nhập  
- Tao realms: click vào ô `Master` ở góc trái chọn `Create Realm` tên là `netbird`
  
![image](https://github.com/diemzero1/netbird/assets/113272767/f92225c9-9d0a-458d-ad23-0ad4e1871037)


  **Tạo User**
- Chọn realms `netbird` rồi click vào tab `Users` và điền tên là `netbird` rồi `Create`

![image](https://github.com/diemzero1/netbird/assets/113272767/b0239b63-e15d-446c-8de5-900a10e27a11)

  
- Chọn tab `Credentials` rồi `Set password` và điền mật khẩu rồi tắt `Temporary` sau đó lưu

![image](https://github.com/diemzero1/netbird/assets/113272767/c832a33d-8ba9-4d22-8c57-a89c467d8a69)


**Tạo Netbird Client**
- Chon tab `Clients`
- Rồi `Create client`
- Client Type: `OpenID Connect`
- Client ID: `netbird-client`
- Việc tạo `netbird-client` sẽ được sử dụng để cấu hình `NETBIRD_AUTH_CLIENT_ID` trong `setup.env`
  
![image](https://github.com/diemzero1/netbird/assets/113272767/720d79bf-475c-400e-997b-20648de0c163)

  
- Sau đó tiếp tục cấu hình như hình sau:

![image](https://github.com/diemzero1/netbird/assets/113272767/e7c58aae-d112-40e7-a595-5a7c08ed199a)


- Tiếp theo với Access settings
- Thay `192.168.15.128` bằng ip của máy và lưu
  
![image](https://github.com/diemzero1/netbird/assets/113272767/58ed0c8a-0c64-460e-9369-319f53910e5b)


**Tạo Client Scope**
- Chọn realms `netbird` rôi chọn tab `Client scopes` và `Create client scope`
- Nhập trường Name là `api`
- Type: `Default`
- Protocol: `OpenID Connect`
- Click `Save`
 
![image](https://github.com/diemzero1/netbird/assets/113272767/b7a42dcd-6890-4b26-b80a-5283ba60bf74)


- Chuyển sang tab `Mappers`
- Click `Configure a new mapper`
- Choose the `Audience` mapping

![image](https://github.com/diemzero1/netbird/assets/113272767/7d972868-e16b-4a82-9f6d-e2928bc446f3)


- Sau đó điền form:  
- Name: `Audience for NetBird Management API`
- Included Client Audience: `netbird-client`
- Add to access token: `On`
- Click `Save`

![image](https://github.com/diemzero1/netbird/assets/113272767/ff6b1ea9-2702-4b2e-beaf-5b4d88afe8eb)


***Add client scope to NetBird client***
- Chọn tab `Clients`
- Choose `netbird-client` from the list
- Switch to `Client scopes` tab
- Click `Add client scope` button
- Choose `api`
- Click `Add` choosing `Default`
- The value `netbird-client` will be used as audience

![image](https://github.com/diemzero1/netbird/assets/113272767/d3d6d12a-4a32-4b96-9318-6b10f20af28a)


***Create a NetBird-Backend client***
- Click `Clients` tab
- Click `Create client` button
- Điền form với nội dung sau và click Next:
- Client Type: `OpenID Connect`
- Client ID: `netbird-backend`
- Client vừa tạo `netbird-backend` được dùng để cấu hình `NETBIRD_IDP_MGMT_CLIENT_ID` trong `setup.env`

![image](https://github.com/diemzero1/netbird/assets/113272767/a4c54fed-8982-453a-9bd3-bf8df84ef769)


![image](https://github.com/diemzero1/netbird/assets/113272767/046d2142-253e-4464-8032-c9b8ac3ec2bc)


- Chọn tab `Credentials` trong `netbird-backend`  
- Copy `client secret` sẽ được dùng để cấu hình `NETBIRD_IDP_MGMT_CLIENT_SECRET` trong `setup.env`

![image](https://github.com/diemzero1/netbird/assets/113272767/32080e7b-51bd-4663-8638-6511b13c8ac6)


***Add view-users role to netbird-backend***
- Click `Clients`
- Chọn `netbird-backend`
- Chọn `Service accounts roles` tab
- Click `Assign roles` button
- Chọn `Filter by clients` và tìm kiếm cho `view-users`

![image](https://github.com/diemzero1/netbird/assets/113272767/bcb2a400-55be-460b-a98e-bfce26e61907)


- Chọn `view-users` và `Assign`

![image](https://github.com/diemzero1/netbird/assets/113272767/84b879fb-438e-4a71-8e3c-35b74efc09dc)


***Bước 4: Run configuaration script***
- Tại thư mục `netbird/infrastructure_files/` chạy lệnh `./configure.sh` bằng command

***Bước 5: Run docker-compos***
```cd artifacts
docker-compose up -d
```

***Bước 6: Check docker logs (Optional)***
```
cd artifacts
docker-compose logs signal
docker-compose logs management
docker-compose logs coturn
docker-compose logs dashboard

```

### Backup
- Lưu các tệp cấu hình  
```
cd netbird/infrastructure_files/artifacts/
mkdir backup
cp docker-compose.yml turnserver.conf management.json backup/

```

- Lưu database  
```
docker compose stop management
docker compose cp -a management:/var/lib/netbird/ backup/
docker compose start management
```

### Issue
***Cấu hình cho truy cập dashboard từ máy khách***
- NETBIRD_DOMAIN sẽ là ip chứ không dùng localhost
  
![image](https://github.com/diemzero1/netbird/assets/113272767/8d298e46-a52c-4b6c-8f97-3b6d99a49d50)


- Sau khi chạy lệnh `./configure.sh` thì cần chỉnh sửa file `docker-compose.yml` thành:

![image](https://github.com/diemzero1/netbird/assets/113272767/cbd504e6-7c2d-4e89-a946-5cadfaed812d)


***LỖi 401 unauthorized***
- Do `api` trong `Client Scopes`
- Khi sử dụng 1 client khác mà không phải `netbird-client` thì cần tạo `Mapper` mới trong `api` bằng cách:
- Chọn tab `Client Scopes`, chọn `api` và chọn `Mapper`
- Chon `Add Mappers`, `By configuaration` rồi chọn `Audience` và điền các giá trị với client tương ứng ở phần tạo client scope trong bước 3
- Lưu và thử lại

***Lỗi kết nối Agent đến Server, có hiển thị lên Dashboard nhưng không Active***
- Sửa cấu hình `signal` trong file `management.json` từ `https` thành `http`

![image](https://github.com/diemzero1/netbird/assets/113272767/099b49a1-0b27-4317-9575-73df6d0bb287)


***Lỗi ERR_CONNECTION_RESET***
- Do thiếu Database Geo location
- B1 Đến thư mục 
  `cd netbird/infrastructure_files/`
  
- B2 Chạy script download 
  `./download-geolite2.sh`

- B3 Copy 2 file `GeoLite2-City.mmdb` và `geonames.db` vào thư mục `netbird/infrastructure_files/artifacts`

- B4 Chạy lệnh command ở thư mục `netbird/infrastructure_files/artifacts`  
```
  docker compose cp geonames.db management:/var/lib/netbird/
  docker compose cp GeoLite2-City.mmdb management:/var/lib/netbird/
  docker compose restart management
```

***Lưu ý***
- Khi chạy `./configure.sh` sẽ tạo ra các file `docker-compose.yml`, `management.json` mới. Nên thực hiện nó trước khi sửa các file trên.
- Nếu `./configure.sh` hoặc `docker-compose` lỗi connect socket thì chạy lệnh  
```
  sudo groupadd docker
  sudo usermod -aG docker $USER
  newgrp docker
```
- Cấu hình netbird-client có thể dùng
  
![image](https://github.com/diemzero1/netbird/assets/113272767/aa1c0722-6e1b-43c0-b388-06ae9bb5eeaf)



### Open-Source Network Security in a Single Platform


![netbird_2](https://github.com/netbirdio/netbird/assets/700848/46bc3b73-508d-4a0e-bb9a-f465d68646ab)


### Key features

| Connectivity                                                                                                                 | Management                                                                                               | Security                                                                                                                              | Automation                                                                                                                               | Platforms                                                                               |
|------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| <ul><li> - \[x] Kernel WireGuard </ul></li>                                                                                  | <ul><li> - \[x] [Admin Web UI](https://github.com/netbirdio/dashboard) </ul></li>                        | <ul><li> - \[x] [SSO & MFA support](https://docs.netbird.io/how-to/installation#running-net-bird-with-sso-login) </ul></li>           | <ul><li> - \[x] [Public API](https://docs.netbird.io/api) </ul></li>                                                                     | <ul><li> - \[x] Linux </ul></li>                                                        |
| <ul><li> - \[x] Peer-to-peer connections </ul></li>                                                                          | <ul><li> - \[x] Auto peer discovery and configuration </ul></li>                                         | <ul><li> - \[x] [Access control - groups & rules](https://docs.netbird.io/how-to/manage-network-access) </ul></li>                    | <ul><li> - \[x] [Setup keys for bulk network provisioning](https://docs.netbird.io/how-to/register-machines-using-setup-keys) </ul></li> | <ul><li> - \[x] Mac </ul></li>                                                          |
| <ul><li> - \[x] Connection relay fallback </ul></li>                                                                         | <ul><li> - \[x] [IdP integrations](https://docs.netbird.io/selfhosted/identity-providers) </ul></li>     | <ul><li> - \[x] [Activity logging](https://docs.netbird.io/how-to/monitor-system-and-network-activity) </ul></li>                     | <ul><li> - \[x] [Self-hosting quickstart script](https://docs.netbird.io/selfhosted/selfhosted-quickstart) </ul></li>                    | <ul><li> - \[x] Windows </ul></li>                                                      |
| <ul><li> - \[x] [Routes to external networks](https://docs.netbird.io/how-to/routing-traffic-to-private-networks) </ul></li> | <ul><li> - \[x] [Private DNS](https://docs.netbird.io/how-to/manage-dns-in-your-network) </ul></li>      | <ul><li> - \[x] [Device posture checks](https://docs.netbird.io/how-to/manage-posture-checks) </ul></li>                              | <ul><li> - \[x] IdP groups sync with JWT </ul></li>                                                                                      | <ul><li> - \[x] Android </ul></li>                                                      |
| <ul><li> - \[x] NAT traversal with BPF </ul></li>                                                                            | <ul><li> - \[x] [Multiuser support](https://docs.netbird.io/how-to/add-users-to-your-network) </ul></li> | <ul><li> -  \[x] Peer-to-peer encryption </ul></li>                                                                                   |                                                                                                                                          | <ul><li> - \[x] iOS </ul></li>                                                          |
|                                                                                                                              |                                                                                                          | <ul><li> - \[x] [Quantum-resistance with Rosenpass](https://netbird.io/knowledge-hub/the-first-quantum-resistant-mesh-vpn) </ul></li> |                                                                                                                                          | <ul><li> - \[x] OpenWRT </ul></li>                                                      |
|                                                                                                                              |                                                                                                          | <ui><li> - \[x] [Periodic re-authentication](https://docs.netbird.io/how-to/enforce-periodic-user-authentication)</ul></li>           |                                                                                                                                          | <ul><li> - \[x] [Serverless](https://docs.netbird.io/how-to/netbird-on-faas) </ul></li> |
|                                                                                                                              |                                                                                                          |                                                                                                                                       |                                                                                                                                          | <ul><li> - \[x] Docker </ul></li>                                                       |
### Quickstart with NetBird Cloud

- Download and install NetBird at [https://app.netbird.io/install](https://app.netbird.io/install)
- Follow the steps to sign-up with Google, Microsoft, GitHub or your email address.
- Check NetBird [admin UI](https://app.netbird.io/).
- Add more machines.

### Quickstart with self-hosted NetBird

> This is the quickest way to try self-hosted NetBird. It should take around 5 minutes to get started if you already have a public domain and a VM.
Follow the [Advanced guide with a custom identity provider](https://docs.netbird.io/selfhosted/selfhosted-guide#advanced-guide-with-a-custom-identity-provider) for installations with different IDPs.

**Infrastructure requirements:**
- A Linux VM with at least **1CPU** and **2GB** of memory.
- The VM should be publicly accessible on TCP ports **80** and **443** and UDP ports: **3478**, **49152-65535**.
- **Public domain** name pointing to the VM.

**Software requirements:**
- Docker installed on the VM with the docker-compose plugin ([Docker installation guide](https://docs.docker.com/engine/install/)) or docker with docker-compose in version 2 or higher.
- [jq](https://jqlang.github.io/jq/) installed. In most distributions
  Usually available in the official repositories and can be installed with `sudo apt install jq` or `sudo yum install jq`
- [curl](https://curl.se/) installed.
  Usually available in the official repositories and can be installed with `sudo apt install curl` or `sudo yum install curl`

**Steps**
- Download and run the installation script:
```bash
export NETBIRD_DOMAIN=netbird.example.com; curl -fsSL https://github.com/netbirdio/netbird/releases/latest/download/getting-started-with-zitadel.sh | bash
```
- Once finished, you can manage the resources via `docker-compose`

### A bit on NetBird internals
-  Every machine in the network runs [NetBird Agent (or Client)](client/) that manages WireGuard.
-  Every agent connects to [Management Service](management/) that holds network state, manages peer IPs, and distributes network updates to agents (peers).
-  NetBird agent uses WebRTC ICE implemented in [pion/ice library](https://github.com/pion/ice) to discover connection candidates when establishing a peer-to-peer connection between machines.
-  Connection candidates are discovered with the help of [STUN](https://en.wikipedia.org/wiki/STUN) servers.
-  Agents negotiate a connection through [Signal Service](signal/) passing p2p encrypted messages with candidates.
-  Sometimes the NAT traversal is unsuccessful due to strict NATs (e.g. mobile carrier-grade NAT) and a p2p connection isn't possible. When this occurs the system falls back to a relay server called [TURN](https://en.wikipedia.org/wiki/Traversal_Using_Relays_around_NAT), and a secure WireGuard tunnel is established via the TURN server. 
 
[Coturn](https://github.com/coturn/coturn) is the one that has been successfully used for STUN and TURN in NetBird setups.

<p float="left" align="middle">
  <img src="https://docs.netbird.io/docs-static/img/architecture/high-level-dia.png" width="700"/>
</p>

See a complete [architecture overview](https://docs.netbird.io/about-netbird/how-netbird-works#architecture) for details.

### Community projects
-  [NetBird installer script](https://github.com/physk/netbird-installer)
-  [NetBird ansible collection by Dominion Solutions](https://galaxy.ansible.com/ui/repo/published/dominion_solutions/netbird/)

**Note**: The `main` branch may be in an *unstable or even broken state* during development.
For stable versions, see [releases](https://github.com/netbirdio/netbird/releases).

### Support acknowledgement

In November 2022, NetBird joined the [StartUpSecure program](https://www.forschung-it-sicherheit-kommunikationssysteme.de/foerderung/bekanntmachungen/startup-secure) sponsored by The Federal Ministry of Education and Research of The Federal Republic of Germany. Together with [CISPA Helmholtz Center for Information Security](https://cispa.de/en) NetBird brings the security best practices and simplicity to private networking.

![CISPA_Logo_BLACK_EN_RZ_RGB (1)](https://user-images.githubusercontent.com/700848/203091324-c6d311a0-22b5-4b05-a288-91cbc6cdcc46.png)

### Testimonials
We use open-source technologies like [WireGuard®](https://www.wireguard.com/), [Pion ICE (WebRTC)](https://github.com/pion/ice), [Coturn](https://github.com/coturn/coturn), and [Rosenpass](https://rosenpass.eu). We very much appreciate the work these guys are doing and we'd greatly appreciate if you could support them in any way (e.g., by giving a star or a contribution).

### Legal
 _WireGuard_ and the _WireGuard_ logo are [registered trademarks](https://www.wireguard.com/trademark-policy/) of Jason A. Donenfeld.

