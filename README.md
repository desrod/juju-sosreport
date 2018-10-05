# jsos - Juju sosreport collector

A quick-and-dirty Ansible playbook to pull sosreports from a deployed juju model. 

### Prerequisites

You'll need a working juju model + controller, and ansible installed on the controller itself. 

You'll also need to install 'jq' on the juju controller so you can parse the resulting json it will output. 

### Getting it going

To generate your inventory file, use the following construct: 

```
juju status --format json | jq -r '.applications[] | select(has("units"))  | .units | to_entries[] | "\(.value."public-address") ansible_host=\(.value."public-address") juju_unit=\(.key)"'
```

You can then copy this into your inventory file and edit as needed (see juju-inventory.yaml here for a working example). 

To run the sosreport collection across your units, just run: 

```
ansible-playbook -v sosreport.yaml -i juju-inventory.yaml -l ceph-osd
```

Note that I've limited the collection to the 'ceph-osd' nodes in this specific case. If you don't pass a filter (-l ceph-osd), then it will collect sosreports from ALL of your units, which may not be what you want. 

After the collection is done, it will fetch those files from the units and put them into /tmp/files/ in your controller. You can change that in the playbook where indicated, if you want a different path. 

### TODO

- auto-upload of the sosreports to the Canonical SFTP Support portal
- Removal of the sosreports from the units after they've been fetched (+checksum) 
- Better regex for accurate file naming conventions of the remote sosreport files

## Authors

* **David A. Desrosiers** - *Initial work*

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to Nick for pushing me to figure out how to do this (sleep--) 
* Automating juju that orchestrates a cluster that automates deployment, oh my! 
