# Properly Delete Node in ProxMox

To remove a Proxmox node that wasn't properly deleted, first ensure the node is powered off and then use the pvecm delnode <nodename> command, followed by removing the node's directory in /etc/pve/nodes/ and restarting the cluster services.

---
  
**1. Power Down the Node:**
 - Before attempting any removal, ensure the problematic node is completely powered off.

**2. Remove the Node from the Cluster:**
 - Identify the Node Name: In the Proxmox GUI, find the name of the node you want to remove.

 - Use `pvecm delnode:**` On a remaining node in the cluster, use the following command, replacing <nodename> with the actual name of the node: pvecm delnode <nodename>.
  
 - **Verify:**` After running the command, check the Proxmox GUI to see if the node is gone. If not, proceed to the next steps.

**3.Remove the Node's Directory:**
  
 - **Locate the Directory:**
 On a remaining node in the cluster, navigate to the directory /etc/pve/nodes/.
 - **Delete the Node's Directory:**
Remove the directory corresponding to the node you want to delete 
  `rm -rf /etc/pve/nodes/<nodename>`.
  
 - **Verify:**
After deleting the directory, check the Proxmox GUI again to see if the node is gone.

**4. Restart Cluster Services (if necessary):**
 - **Restart Services:**
If the node still persists in the GUI, try restarting the Proxmox cluster services: 
`systemctl restart pve-cluster.service.`

 - **Restart Corosync:**
You may also need to restart corosync: 
`systemctl restart corosync`

**5. Check for leftover entries:**
 - **Check /etc/pve/storage.cfg:** 
Make sure there are no entries for the deleted node in the storage configuration file.
 - **Check /etc/pve/.members:** 
Verify that the node is not listed in the cluster members file.
 - **Check /etc/pve/corosync.conf:** 
Make sure the node is not listed in the corosync configuration file.

> **6. Troubleshooting:**
{.is-info}

 - **Quorum Issues:**
If you encounter quorum issues, try setting the expected number of nodes with pvecm expected 1.

 - **Permissions:** 
If you encounter permission issues, ensure you are logged in as root and that the /etc/pve directory is not owned by a user other than root.

 - **HA Configuration:** Check your cluster HA configuration for any weird entries or ghost machines.
  
 - **Restart HA services:** 
`systemctl restart pve-ha-lrm.service pve-ha-crm.service`
  
 - **Check for same package versions:** 
Ensure all nodes are running the same Proxmox package versions.
