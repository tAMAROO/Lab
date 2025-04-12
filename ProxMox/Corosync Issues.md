# Corosync Issues
After refreshing the cluster I was having issues with corosync failing to start:

---

**Error:**
This is the error I got while trying to start the service.
>[MAIN] Cannot chdir to state directory /var/lib/corosync. No such file or directory
{.is-danger}


**Solution:**

I created the folder /var/lib/corosync and the service started and fixed the issue:

> `mkdir /var/lib/corosync`
