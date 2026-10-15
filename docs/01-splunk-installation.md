[01-splunk-installation.md](https://github.com/user-attachments/files/33495840/01-splunk-installation.md)
# 01. Splunk Enterprise installation (Ubuntu Server 24.04, `.deb`)

Lab VM: VirtualBox, 4 vCPU, 8 GB RAM, 100 GB disk, Bridged network.

## 1. Prepare Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y wget curl tar net-tools tmux
ip a        # note the VM IP
df -h /     # confirm full disk size
```

If `df -h /` shows less than the disk you allocated (LVM default), extend it:

```bash
sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

## 2. Download and install the `.deb`

Get the Linux `.deb` link from the Splunk download page (free account required).

```bash
cd /tmp
wget -O splunk.deb "<link from the Splunk download page>"
sudo dpkg -i splunk.deb
ls -ld /opt/splunk        # owner should be splunk:splunk
```

## 3. Create the admin account

Use a strong password of your own. Never commit this file or the password.

```bash
sudo tee /opt/splunk/etc/system/local/user-seed.conf > /dev/null <<'EOT'
[user_info]
USERNAME = admin
PASSWORD = <your-strong-password>
EOT
sudo chown splunk:splunk /opt/splunk/etc/system/local/user-seed.conf
```

## 4. First start as the `splunk` user, then stop

```bash
sudo -u splunk /opt/splunk/bin/splunk start --accept-license --answer-yes --no-prompt
sudo -u splunk /opt/splunk/bin/splunk stop
```

## 5. Run as a systemd service

The `.deb` registers an init.d service by default, so disable it first.

```bash
sudo /opt/splunk/bin/splunk disable boot-start 2>/dev/null || true
sudo chown -R splunk:splunk /opt/splunk
sudo /opt/splunk/bin/splunk enable boot-start -systemd-managed 1 -user splunk -group splunk
sudo systemctl daemon-reload
sudo systemctl start Splunkd
sudo systemctl status Splunkd --no-pager
```

Expected: `active (running)` and `enabled`. Confirm it does not run as root:

```bash
ps -o user,pid,cmd -C splunkd | head -3
```

## 6. Log in

Open `http://<VM-IP>:8000` and sign in with the admin account. Allow 1 to 3 minutes after start.

If `ufw` is active:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 8000/tcp
```

## 7. Snapshot

Take a VirtualBox snapshot here (`splunk-clean-install`) before loading any dataset.
