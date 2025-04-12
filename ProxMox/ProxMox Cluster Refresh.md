# ProxMox Cluster Refresh
To completely remove a Proxmox cluster configuration and start fresh, you need to stop the cluster services, clear the cluster configuration files, and then restart the services.

---

**1. Stop the Proxmox Cluster Services:**
 - Stop the pve-cluster and corosync services:
`systemctl stop pve-cluster`
 `systemctl stop corosync`


**2. Clear Cluster Configuration Files:** 
 - start pmxcfs in local mode.
    `pmxcfs -l`
**Remove the cluster configuration files.**
    `rm /etc/pve/corosync.conf`
    `rm -r /etc/corosync/*`
    `rm /var/lib/corosync/*`


**3. Restart the Cluster Services:**
 - Kill the local mode pmxcfs process.
`killall pmxcfs`

 - Start the pve-cluster service.
`systemctl start pve-cluster`


> **Important Notes:**
{.is-info}


 - **Data Loss:**
This process will remove the cluster configuration, so ensure you have backed up any critical data or VMs before proceeding.

 - **Node Removal:**
If you're removing a node from the cluster, you should first shut down the node and then use pvecm delnode <node_name> to remove it from the cluster configuration.

 - **Reinstalling a Node:**
If you want to add a node back to the cluster, you'll likely need to reinstall the node and then reconfigure it to join the cluster.
