RHEV-scripts
============

	Virtual machines backup script for RHEV 3
24/03/2014/5 Comments/in Red Hat Enterprise Virtualization (RHEV) English /by Antoine Hordez

Red Hat Enterprise Virtualization 3 (RHEV) comes with a great API that makes it easy to write Python scripts or Java programs for administrative tasks. The full documentation is available here : https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Virtualization/3.3/html/Developer_Guide/index.html

A classical need is to automate the creation of virtual machine cold backups on an export domain. This can be easily done by using the Python API.

First you need to install the RHEV Software Development Kit :
$ yum install rhevm-sdk

You can now write a Python script using the ovirtsdk (RHEV SDK) :
#!/usr/bin/python

from ovirtsdk.api import API

Let’s declare a few constants :
RHEV_URL = "https://127.0.0.1"
RHEV_USERNAME = "admin@internal"
RHEV_PASSWORD = "ThePassword"

Then try to connect to RHEV :
api = API ( url=RHEV_URL,
            username=RHEV_USERNAME,
            password=RHEV_PASSWORD,
            ca_file="/etc/pki/ovirt-engine/ca.pem")

print "Connected to %s successfully!" % api.get_product_info().name

We can display information about the virtual machine “vm1″ :
vm = api.vms.get(name="vm1")

print "VM name : %s" % (vm.name)

vm_status = vm.status.state
print "Status : %s" % (vm_status)

vm_cluster = api.clusters.get(id=vm.cluster.id)
print "Cluster : %s" % (vm_cluster.get_name())

vm_dc = api.datacenters.get(id=vm_cluster.data_center.id)
print "Datacenter : %s" % (vm_dc.get_name())

Another example, stop a virtual machine :
if api.vms.get(name="vm1").status.state != 'down':
    api.vms.get(name="vm1").shutdown()
    print 'Waiting for VM to reach Down status'
    while api.vms.get(name="vm1").status.state != 'down':
        sleep(1)
    print 'OK'
else:
    print "VM already down"

As you can see it’s really easy to get information about a virtual machine like the cluster, the datacenter, the state, etc.

Now let’s have a look at the virtual machine full backup scenario. A possible process could be to :

    Collect the virtual machine name to backup
    Connect to RHEV
    Save the initial state of the virtual machine (up, down, etc.)
    Ensure that the export domain is not mounted on any other RHEV datacenter
    Mount the export domain of the virtual machine’s datacenter
    Stop the virtual machine (if ‘up’)
    Export the virtual machine to the export domain
    Restore the initial state of the virtual machine (previously saved)
    Unmount the export domain
    Disconnect from RHEV

All of these operations can be done using the Python API.
You can find a full example script on our GitHub page https://github.com/clevernet/RHEV-scripts/blob/master/backup-vm.py or below :
