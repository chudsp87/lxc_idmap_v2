# lxc_idmap_v2
Little python utility to provide the text needed for /etc/lxc/<ctid>.conf, /etc/subuid, and /etc/subgid to map user and group ids mappings for unprivileged lxcs.

As used below: 
- lxc_uid = container user id
- lxc_gid = container group id
- host_uid = host (proxmox) user id
- host_gid = host (proxmox) group id
  

<h2>Syntax</h2>

usage: `lxc_idmap_v2.py [-h] [-u [USER]] [-g [GROUP]] [lxc_uid[:lxc_gid][=host_uid[:host_gid]] ...]`

<b>All arguments can be used as many times as needed, but must include the -u/-g flag for each flagged argument.</b>

Ex. `-u 1000 1234 -g 1000` != `-u 1000 -u 1234 -g 1000`  
Note: above is still valid syntax, but the `1234` will be interpretted as unflagged arg, i.e. -> `1234:1234=1234:1234`

<em>Don't duplicate use of the same id</em>, I don't have any handling to skip over that, so the config will be invalid...


<h3>UID|GID Mappings (no flag args)</h3>

`lxc_uid[:lxc_gid][=host_uid[:host_gid]]` (no flag)

- *Only the container user's id `<lxc_uid>` is required.*
- if no container group id `<lxc_gid>` then it will be set to the user id
  - As such: `./lxc_idmapper 1000` is equivalent to `./lxc_idmapper 1000:1000`   
- if no host user id is provided, it will be set to the container user id.
  - As such: `./lxc_idmapper 1000` is equivalent to `./lxc_idmapper 1000=1000`  
- if no host group id is provided, it will be set to the container group id.
  - As such: `./lxc_idmapper 1000:1234` is equivalent to `./lxc_idmapper 1000:1234=1000:1234`

- Separate multiple id mappings with a `<space>`
 
<h2>Flagged Arguments</h2>
Sometimes this behavior of creating a group id mapping for every user id mapping (or vice versa) is undesired. Utilize the -u/--user and -g/--group to limit mapping to only the user or group identified. 

<h3>-u | --user</h3>

Usage: `-u lxc_id` or `--user lxc_id=host_id`
Similar to above, in that only the container id is required. If not provided, the host id will default to the same value of the container id. 

<h3>-g | --group</h3>

Usage: `-g lxc_id` or `--group lxc_id=host_id`
Similar to above, in that only the container id is required. If not provided, the host id will default to the same value of the container id. 

<h2>Example Usage and Output</h2>

```bash
./lxc_idmap_v2.py 1000 # equivalent to ./lxc_idmap_v2 1000:1000 or ./lxc_idmap_v2 1000:1000=1000:1000

# Add to /etc/pve/lxc/<container_id>.conf:
lxc.idmap: u 0 100000 1000
lxc.idmap: u 1000  1000 1
lxc.idmap: u 1001 101001  64535
lxc.idmap: g 0 100000 1000
lxc.idmap: g 1000  1000 1
lxc.idmap: g 1001 101001  64535

# Add to /etc/subuid:
root:1000:1

# Add to /etc/subgid:
root:1000:1
```

```bash
./lxc_idmap_v2.py 1000:9876

# Add to /etc/pve/lxc/<container_id>.conf:
lxc.idmap: u 0 100000 1000
lxc.idmap: u 1000  1000 1
lxc.idmap: u 1001 101001  64535
lxc.idmap: g 0 100000 9876
lxc.idmap: g 9876  9876 1
lxc.idmap: g 9877 109877  55659

# Add to /etc/subuid:
root:1000:1

# Add to /etc/subgid:
root:9876:1
```

```bash
./lxc_idmap_v2.py 1000:567=990:213

# Add to /etc/pve/lxc/<container_id>.conf:
lxc.idmap: u 0 100000 1000
lxc.idmap: u 1000  990 1
lxc.idmap: u 1001 101001  64535
lxc.idmap: g 0 100000 567
lxc.idmap: g 567  213 1
lxc.idmap: g 568 100568  64968

# Add to /etc/subuid:
root:990:1

# Add to /etc/subgid:
root:213:1
```

```bash
/lxc_idmap_v2.py 564:564=812 -u 444=1230 -g 909 -g 7777

# Add to /etc/pve/lxc/<container_id>.conf:
lxc.idmap: u 0 100000 444
lxc.idmap: u 444  1230 1
lxc.idmap: u 445  100445 119
lxc.idmap: u 564  812 1
lxc.idmap: u 565 100565  64971
lxc.idmap: g 0 100000 564
lxc.idmap: g 564  564 1
lxc.idmap: g 565  100565 344
lxc.idmap: g 909  909 1
lxc.idmap: g 910  100910 6867
lxc.idmap: g 7777  7777 1
lxc.idmap: g 7778 107778  57758

# Add to /etc/subuid:
root:1230:1
root:812:1

# Add to /etc/subgid:
root:564:1
root:909:1
root:7777:1
```
