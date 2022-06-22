# clash-hosting-updater

Before your start, make sure you have successfully run clash with the help of the [documentation](https://github.com/Dreamacro/clash/wiki).

## Getting Started

1. Download `script.py` and `updater.py` to `/etc/clash`

2. Modify `config.yaml`

    # The updater will reload the configuration using the RESTful API
    external-controller: :9090
    mode: Script

    # https://github.com/Dreamacro/clash/releases/tag/premium
    script:
        path: ./script.rules.py

3. Configure [clash systemd](https://github.com/Dreamacro/clash/wiki/clash-on-a-daemon#systemd)

    [Unit]
    Description=Clash daemon, A rule-based proxy in Go.
    After=network.target

    [Service]
    Type=simple
    Restart=always
    ExecStart=/opt/gopath/bin/clash -d /etc/clash
    ExecReload=/bin/curl -X 'PUT' 'http://127.0.0.1:9090/configs?force=true'  \
                         -d '{"path": "/etc/clash/config.yaml"}'

    [Install]
    WantedBy=multi-user.target

4. Install python3, minimum supported 3.6.8

5. Create the systemd configuration file at `/etc/systemd/system/updater.service`

    [Unit]
    Description=Clash hosting updater daemon, A rule-based script builder in Python.
    Requires=clash.service

    [Service]
    Type=simple
    Restart=always
    ExecStart=/usr/bin/python3 -u /etc/clash/updater.py DefaultPolicies URL
    WorkingDirectory=/etc/clash
    ExecReload=/bin/kill -s HUP $MAINPID
    KillMode=process

    [Install]
    WantedBy=multi-user.target

`updater.py` takes two parameters, in order, `DefaultPolicies` and `URL`.
The updater checks for the existence of rule-policies in `config.yaml`
(proxy-groups only) and uses `DefaultPolicies` when they do not exist.
The clash hosting address should follow the [URL Scheme](https://docs.cfw.lbyczf.com/contents/urlscheme.html).

Launch updater on system startup with:

    $ systemctl enable updater

Launch updater immediately with:

    $ systemctl start updater

Check the health and logs of updater with:

    $ systemctl status updater
    $ journalctl -xe

You can change the service name to your needs.