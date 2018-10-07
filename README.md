# jsos - Juju sosreport collector

A quick-and-dirty Ansible playbook to pull sosreports from units running within a deployed juju model and auto-upload those to the Canonical Support SFTP portal.

### Prerequisites and Dependencies

You'll need a working juju model + controller, and ansible installed on the controller itself. 

You'll also need to install `jq` on the juju controller so you can parse the resulting json it will output and (at the moment) `sshpass` to use sftp non-interactively. I will add some work later to remove the `sshpass` dependency, as not all clients will be able to install this.

### Building your Ansible Inventory file with juju and jq

To generate your inventory file, use the following construct: 

```
juju status --format json | jq -r '.applications[] | select(has("units")) | .units | to_entries[] | "\(.value."public-address") ansible_host=\(.value."public-address") juju_unit=\(.key)"'

```

You can then copy this into your inventory file and edit as needed (see `juju-inventory.yaml` in this repository for a working example). 

Here's a small sample of what that looks like: 

```
[ceph-mon]
192.168.100.211 ansible_host=192.168.100.211 juju_unit=ceph-mon/0
192.168.100.212 ansible_host=192.168.100.212 juju_unit=ceph-mon/1
192.168.100.216 ansible_host=192.168.100.216 juju_unit=ceph-mon/2

[ceph-osd]
192.168.100.203 ansible_host=192.168.100.203 juju_unit=ceph-osd/0
192.168.100.204 ansible_host=192.168.100.204 juju_unit=ceph-osd/1
192.168.100.205 ansible_host=192.168.100.205 juju_unit=ceph-osd/2
```

### How to run the collection across scoped units

To run the sosreport collection across your units, just run: 

```
ansible-playbook -v jsos.yaml -i juju-inventory.yaml -l ceph-osd
```

Note that I've limited the collection to the `ceph-osd` nodes in this specific case. If you don't pass a filter (`-l ceph-osd`), then it will collect sosreports from ALL of your units, which or may not be what you want.

There is a very specific reason the inventory file is constructed in this way. Many times, units will be clustered across multiple physical (or virtual) nodes. You may want to collect from ceph-osd, but those hosts may also host other units. 

The hosts will appear multiple times in different host groups, but the targeted collection will name the resulting sosreports based on your filter, not on the host's DNS name or IP address. This helps keep the sosreports named appropriately, even if multiple sosreports exist for the same physical node running the unit.

After the collection is done, the playbook will copy those files from the units and put them into /tmp/sosreport-{case_id}/ in your controller. You can change that in the playbook's `vars.yaml` file where indicated, if you want to store them in different path on your controller. 

From there, the collected sosreports will be uploaded to the support portal, using the support user, password and SSH key file to authenticate. They will land in a directory named for the `case_id`, with files depositied inside that directory.

### TODO Items

- Add support for public-address containing FQDN of the units
- Auto-generate the inventory file with correct host-group headers
- Remove dependency for sshpass to log into SFTP non-interactively
- Implement a cleaner solution to importing the Support Portal SSH key

### Completed Items

- Better regex for accurate file naming conventions of the remote sosreport files
- Moved all private vars into vars.yaml, instead of the playbook itself
- auto-upload of the sosreports to the Canonical SFTP Support portal
- Removal of the sosreports from the units after they've been fetched (+checksum)

## Authors

* [David A. Desrosiers](mailto:david.desrosiers@canonical.com)

## Acknowledgments

* A dire client need initially drove the idea to creat this tool
* Hat tip to Nick N. for pushing me to figure out how to do this clealy (who needs sleep anyway!) 
* Automating juju that orchestrates a cluster that automates deployment, oh my! 
