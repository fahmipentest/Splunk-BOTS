# 04. Command cheat sheet

## Splunk service (systemd)

```bash
sudo systemctl start Splunkd
sudo systemctl stop Splunkd
sudo systemctl restart Splunkd
sudo systemctl status Splunkd --no-pager
sudo systemctl enable Splunkd      # start at boot
```

Do not run `sudo /opt/splunk/bin/splunk start|restart`. It starts Splunk as root.

## Health checks

```bash
sudo systemctl is-active Splunkd
ps -o user,pid,cmd -C splunkd | head -3     # USER should be splunk
sudo ss -tlnp | grep -E '8000|8089'
df -h /
```

## Logs

```bash
sudo journalctl -u Splunkd -n 30 --no-pager
sudo tail -50 /opt/splunk/var/log/splunk/splunkd.log
sudo grep -i error /opt/splunk/var/log/splunk/splunkd.log | tail -20
```

## tmux

```bash
tmux new -s bots        # new session
# Ctrl+b then d         # detach
tmux ls                 # list sessions
tmux attach -t bots     # re-attach
```

## SPL starters

```spl
| tstats count where index=botsv1 OR index=botsv2 OR index=botsv3 by index
index=botsv3 earliest=0 | stats count by sourcetype | sort - count
index=botsv3 earliest=0 | stats count by host | sort - count
index=botsv3 earliest=0 sourcetype=<sourcetype> | timechart span=1h count
```
