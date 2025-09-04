# Local Server Installation
Get the latest version of source code, at this moment its `25.8.0`

```sh
wget https://github.com/actualbudget/actual/archive/refs/tags/v25.8.0.tar.gz
```

Extract it to folder of choice
```sh
tar -xzf v25.8.0.tar.gz -C ~/.actual
```

Change folder name to server
```sh
mv actual-25.8.0 server
```

cd to it
```sh
cd ~/.actual/server
```

install dependencies, this installation requires node and yarn installed, see `nodejs.md` notes
```sh
yarn install
```

Build the server, this takes some time....
```sh
yarn build:server
```

Start the server
```sh
yarn start:server
```

## Configuration
To automatically shoot up the server on reboot, we need to setup systemd service

Create a `.service` file and place it in `/etc/systemd/system`

```bash
[Unit]
Description=Actual Budget
After=network.target

[Service]
WorkingDirectory=/home/yosang/.actual
ExecStart=yarn start:server
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Reload the daemon `systemctl daemon-reload`

Enable and start `sudo systemctl enable --now actual.service`

# Dcoker image
Pull from the actual official repo

```sh
docker pull actualbudget/actual-server
```

Run it

```sh
docker run --pull=always --restart=unless-stopped -d -p 5006:5006 -v /home/yosang/.sync/.db/actual:/data:Z actualbudget/actual-server:latest
```

# Updating
Whenever an update is released, we stop and delete the container and deploy the image again, with `--pull=always`. This will pull the `diff` from the remote and update our `cached` image.