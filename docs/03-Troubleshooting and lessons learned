# 03. Troubleshooting and lessons learned

Real issues from building this lab.

| Problem | Cause | Fix |
|---|---|---|
| `enable boot-start` fails: "Initd script exists" | The `.deb` registers Splunk as an init.d service | `splunk disable boot-start`, then re-enable with `-systemd-managed 1` |
| Splunk running as `root` | Started with `sudo .../splunk start` | Stop it, `chown -R splunk:splunk /opt/splunk`, run via systemd as `splunk` |
| `splunk restart` refuses to run | Recent Splunk versions deprecate running as root | Use `systemctl restart Splunkd` |
| `ps aux \| grep "splunkd -p 8089"` shows nothing although Splunk runs | Under systemd the process command line differs | Check with `systemctl status Splunkd` and `ps -o user,pid,cmd -C splunkd` |
| Splunkbase install: "Unable to login with given credentials" | Splunk asks for the splunk.com **username** (email may not work), and accounts created through Google/SSO have no password | Check the username in the splunk.com profile or reset the password. Workaround: download on the laptop and use **Install app from file** <!-- TODO: note the actual cause in this lab --> |
| Large dataset download interrupted | SSH session dropped | Run inside `tmux` and use `wget -c` to resume |
| `index=botsvX` returns nothing | Wrong time range, nested folder, wrong ownership, or disk full during extract | Use **All time** and `earliest=0`. Check `ls /opt/splunk/etc/apps/botsvX_data_set/`. Fix ownership. Check `df -h /` |
| Version gap (datasets built on Splunk 6.x/7.x, lab runs 10.6) | Dataset READMEs warn other versions may not work | Check `splunkd.log` for bucket errors. If a dataset does not load, test an older Splunk (9.x) <!-- TODO: record the actual result in this lab --> |

## Takeaways

- Never run a SIEM as root. Use a dedicated service account.
- Know which process manager owns a service (init.d or systemd) before debugging it.
- Verify downloads (MD5 or `gzip -t`) before extracting multi-gigabyte archives.
- Snapshot the VM at known-good points.
- Pre-indexed datasets must be extracted so `default/` and `var/` sit directly under the app folder.
