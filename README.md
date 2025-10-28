# x-ui install script

This is an opionated installation procedure for x-ui using ansible

This playbook:
- Installs some tools like htop, iperf3, etc...
- Installs and configures certbot for automatic cert renewal
- Installs docker with docker compose plugin
- Installs x-ui from alireza0
- Automatically configures x-ui: script updates panel port, panel url, admin password, initial xray-config with my personal preferences like blocking connection to Russia, etc
- Configures telegram notifications

# Usage

First of all, install ansible and external roles:

```
ansible-galaxy role install geerlingguy.certbot
ansible-galaxy role install geerlingguy.docker
```

Change variables inside of `playbook.yml`. Especially `xui_hostname` and certbot domain and admin email:

| variable | description |
|----------|-------------|
| certbot_admin_email | your email address |
| certbot_certs.email | your email address |
| certbot_certs.domains | your domain |
| xui_hostname | your domain |
| socks_username (inside secrets file) | username for socks5 proxy |
| socks_password (inside secrets file) | password for socks5 proxy |

Change inventory in `inventory.ini`: fill IP address of your VPS

Set up secrets for telegram:
1. Copy `secrets_file.enc.template` to `secrets_file.enc`
2. Fill you telegram bot token - can be obtained from @botfather
3. Fill you id - can be obtained from @userinfobot
4. Execute `ansible-vault encrypt secrets_file.enc` and set your password

Run playbook:

```
ansible-playbook -i inventory.ini playbook.yaml -e @secrets_file.enc --ask-vault-pass --tags="requirements,configure-xray"
```

You will be asked for password. Use password for encrypted secrets storage

Panel url and credentials will be shown in the output of the ansible script:

![credentials](./images/credentials.png)

Panel should be fully ready to configure inbounds. Select your desired domain for masking and start using VPN :)

# Inbounds configuration

Configure inbounds according to your preferences. Some tips while configuring vless:
- Use reality
- Set `destination` to `my.domain.com:8001` - nginx should listen on this port and provide a valid TLS certificate (self-stealing)
- DO NOT SET DESTINATION TO `my.domain.com:443` - you will get packet loop
- Set `SNI` to current `my.domain.com`

Check your domain after inbound configuration (`https://my.domain.com` - https, default ssl port)
Working page with a website from `nginx_upstream_website` should be able to load

If nothing loads, troubleshoot your setup. Otherwise, feel free to create clients

# Outbound configuration

By default, all connections will exit from your node. Configure Cloudflare WARP if necessary
