# Troubleshooting log

Each problem I hit during the install and the lab, with cause and fix.
Services run as systemd units named `devstack@<service>`.

| # | Problem | Cause | Fix |
|---|---|---|---|
| 1 | Installed Ubuntu **Desktop** instead of Server | Wrong ISO | Deleted the VM, recreated it with the `ubuntu-22.04.5-live-server` ISO |
| 2 | Hard to type commands in the VM window | AZERTY keyboard not handled well by the VirtualBox window | Connected over SSH from PowerShell (`ssh user@192.168.56.105`) |
| 3 | `Failed unmounting /cdrom` at end of install | ISO still in the virtual drive | Ejected the ISO from the VirtualBox Devices menu, then rebooted |
| 4 | `stable/2024.2` branch not found | Branch does not exist in the DevStack repo | `git checkout unmaintained/2024.1` |
| 5 | `stack.sh` failed: `Connection refused` to opendev.org | Network drop while cloning | `rm -rf /opt/stack/keystone` (incomplete clone) and rerun `./stack.sh` |
| 6 | Apache error `No module named encodings` | Global `WSGIPythonHome` pointed to DevStack's virtualenv | Removed it, added `python-home=/opt/stack/data/venv` to `WSGIDaemonProcess` (`horizon.conf` and DevStack's Apache template) |
| 7 | Neutron: `No tenant network is available for allocation` | `neutron.ml2_geneve_allocations` table empty | `sudo systemctl restart devstack@q-svc` (table refilled to 65,536 rows); set `NEUTRON_CREATE_INITIAL_NETWORKS=False` and created `public`, `private` and `router1` manually |
| 8 | Glance: 503 on `/image`, then 500 on image upload | uWSGI socket missing, then a stale `endpoint_id` in the running process | `sudo systemctl restart devstack@g-api`, then uploaded CirrOS manually with `openstack image create` |
| 9 | Horizon: 403 Forbidden | Apache could not traverse `/opt/stack` | `sudo chmod o+x /opt/stack` |
| 10 | Instance in Error: `No valid host was found` | Compute node had not registered its resources in Placement (empty inventory, `ResourceProviderSyncFailed`) | Restarted `devstack@placement-api`, then `devstack@n-cpu`, then `nova-manage cell_v2 discover_hosts`; inventory showed VCPU, MEMORY_MB, DISK_GB; relaunched the instance |
| 11 | Instance boot log: repeated failures reaching `169.254.169.254` | `q-ovn-metadata-agent` had failed | Restarted `devstack@q-ovn-metadata-agent`, soft-rebooted the instance (DHCP lease and login prompt appeared) |

## Useful checks

```bash
sudo systemctl status devstack@q-svc devstack@g-api devstack@n-cpu devstack@placement-api
source ~/devstack/openrc admin admin
openstack compute service list
openstack image list
openstack resource provider list
openstack server list
```
