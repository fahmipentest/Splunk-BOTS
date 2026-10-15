# 02. BOTS datasets setup (v1, v2, v3)

Each dataset is a pre-indexed Splunk app. It is extracted into `/opt/splunk/etc/apps/` and creates its own index. No forwarders and no volume-based license limits are involved.

| Version | Index | Size (compressed) | Notes |
|---|---|---|---|
| BOTS v1 | `botsv1` | about 6.1 GB (full) | An attack-only version (about 135 MB) also exists |
| BOTS v2 | `botsv2` | 16.4 GB (full) | Attack-only (3.2 GB) exists. Install only one of the two |
| BOTS v3 | `botsv3` | about 320 MB | |

Source repositories: `github.com/splunk/botsv1`, `botsv2`, `botsv3`. Use the current links from each README if a download fails.

## General procedure

Run long downloads inside `tmux` so an SSH drop does not interrupt them.

```bash
tmux new -s bots
df -h / /tmp          # check free space first
```

Extract as the `splunk` user so file ownership is correct:

```bash
sudo -u splunk tar -xzf /tmp/<file>.tgz -C /opt/splunk/etc/apps/
ls /opt/splunk/etc/apps/<dataset_folder>/     # expect: default/ and var/ directly inside
rm /tmp/<file>.tgz
sudo systemctl restart Splunkd
```

## BOTS v1

```bash
cd /tmp
wget -c https://s3.amazonaws.com/botsdataset/botsv1/splunk-pre-indexed/botsv1_data_set.tgz
gzip -t botsv1_data_set.tgz && echo "FILE OK"
sudo -u splunk tar -xzf botsv1_data_set.tgz -C /opt/splunk/etc/apps/
ls /opt/splunk/etc/apps/botsv1_data_set/
rm botsv1_data_set.tgz
sudo systemctl restart Splunkd
```

## BOTS v2 (full dataset)

```bash
cd /tmp
wget -c https://s3.amazonaws.com/botsdataset/botsv2/botsv2_data_set.tgz
md5sum botsv2_data_set.tgz     # expect: fd2673726c96e97a39fc03119d6686c6
sudo -u splunk tar -xzf botsv2_data_set.tgz -C /opt/splunk/etc/apps/
ls /opt/splunk/etc/apps/botsv2_data_set/
rm botsv2_data_set.tgz
sudo systemctl restart Splunkd
```

## BOTS v3

```bash
cd /tmp
wget -c https://botsdataset.s3.amazonaws.com/botsv3/botsv3_data_set.tgz
md5sum botsv3_data_set.tgz     # expect: d7ccca99a01cff070dff3c139cdc10eb
sudo -u splunk tar -xzf botsv3_data_set.tgz -C /opt/splunk/etc/apps/
ls /opt/splunk/etc/apps/botsv3_data_set/
rm botsv3_data_set.tgz
sudo systemctl restart Splunkd
```

## Verify

In Search & Reporting, set Time Range to **All time**:

```spl
| tstats count where index=botsv1 OR index=botsv2 OR index=botsv3 by index
```

```spl
index=botsv1 earliest=0 | stats count by sourcetype | sort - count
```

Repeat the second query for `botsv2` and `botsv3`.

## Add-ons

Each dataset README lists the apps and add-ons it was built with. Datasets were created on older Splunk versions (v1: 6.5.2, v2: 7.2.1, v3: 7.1.7), and the READMEs note that other versions may or may not work. This lab runs Splunk 10.6, so I pick add-on versions compatible with Splunk 10 and skip any that are unavailable.

| Dataset | Main add-ons |
|---|---|
| v1 | Microsoft Windows, Sysmon, TA-Suricata, Splunk Stream, Fortinet FortiGate, Tenable, URL Toolbox |
| v2 | CIM, Windows, Sysmon, Suricata, Stream, Palo Alto Networks, Apache, Microsoft IIS, Microsoft Cloud Services, Symantec Endpoint Protection, OSquery, Unix and Linux, URL Toolbox, Base64, Security Essentials and others from the README |
| v3 | CIM, Windows, Sysmon, AWS, Office 365, Microsoft Cloud Services, Symantec Endpoint Protection, Cisco ASA, Unix and Linux, Stream, osquery, Code42, Security Essentials |

Install via **Apps → Find More Apps**, or download from Splunkbase and use **Apps → Manage Apps → Install app from file**. Missing add-ons usually only affect automatic field extraction, which can be handled in SPL with `rex` or `spath`.
