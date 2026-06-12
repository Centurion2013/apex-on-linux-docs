# Minimal Linux Installation

To optimize resource consumption, consider choosing a **Minimal** configuration during Linux setup.

## Useful packages to add on Minimal

Install:
```console
sudo dnf -y install vim nano wget curl unzip tar lsof bind-utils net-tools firewalld openssh-server rsync 
```

Then enable SSH/firewall as needed:
```console
sudo systemctl enable --now sshd
sudo systemctl enable --now firewalld
```

## Cockpit, the Linux Web Console

Basic install Cockpit:
```console
sudo dnf install cockpit
```

Additional modules:
```console
sudo dnf install cockpit-storaged
sudo dnf install cockpit-packagekit
```

If you plan use docker/podman:
```console
sudo dnf install cockpit-podman
```

Enable and start it:
```console
sudo systemctl enable --now cockpit.socket
```

Check status:
```console
systemctl status cockpit.socket
```

Add firewall rule:
```console
sudo firewall-cmd --permanent --add-service=cockpit
sudo firewall-cmd --reload
```

Connect from browser:
```console
https://<your-server-ip>:9090
```

Check specifically whether Cockpit is allowed:
```console
sudo firewall-cmd --query-service=cockpit
```

Expected:
```console
yes
```
