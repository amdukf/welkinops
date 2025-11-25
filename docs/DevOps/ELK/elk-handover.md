📦 ELK Cluster Handover Document
Welcome to the ELK cluster! 🎉 
This document will guide you through important health checks, monitoring commands, and installation tips to maintain and troubleshoot the cluster.
🔎 Common Health and Monitoring Commands
Use the following commands to check the health and status of the cluster:
📋 Task Manager Check
GET .kibana_task_manager/_search?size=1

❤️ Cluster Health
GET _cluster/health?pretty
GET /_cluster/health

📈 Node Statistics
GET _nodes/stats?pretty

🗂️ Shard Information
GET /_cat/shards?v
GET /_cat/shards?v&h=index,shard,prirep,state,node,unassigned.reason

⚙️ Cluster Settings
GET /_cluster/settings

🖥️ Node Overview
GET /_cat/nodes?v

🗃️ Allocation Information
GET /_cat/allocation?v

🛠️ Recovery and Reroute Commands
If needed, you can manually retry failed shard allocations:
POST /_cluster/reroute?retry_failed=true

(Execute this twice if necessary.)
🧹 Additional Useful Commands
Here are more daily-use commands:
Cluster Health:
GET /_cluster/health?pretty

Node List:
GET /_cat/nodes?v

Indices List:
GET /_cat/indices?v

Shards Overview:
GET /_cat/shards?v

Allocation Overview:
GET /_cat/allocation?v

Specific Index Health (example: filebeat):
GET /_cluster/health/filebeat-sw-8.14.0-2024.07.29?pretty
GET /_cat/shards/filebeat-sw-8.14.0-2024.07.29?v

🛠️ Installation Important Step
To ensure Elasticsearch works properly on your server, configure the maximum number of memory map areas:
Set the value temporarily:
sudo sysctl -w vm.max_map_count=262144

Make the setting permanent:
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf


🔍 Elasticsearch Useful CLI Commands
These commands help inspect and manage disk usage and indices across your Elasticsearch nodes.
—
📦 Cluster Allocation (Disk Space & Shards)
GET _cat/allocation?v=true&h=shards,disk.indices,disk.used,disk.avail,disk.total,node
Description: Shows how many shards are on each node and the disk space usage (used, total, available). Helpful for identifying uneven shard distribution or full disks.
—
📑 List All Indices
GET _cat/indices?v
Description: Displays a summary of all indices including health, status, and size. Use this to get a quick overview of everything in the cluster.
—
📅 Filter Indices by Date (May 1st, 2025)
GET _cat/indices/*2025.05.1*?v
Description: Lists all indices matching the pattern for May 1st, 2025. Useful when reviewing time-series indices.
—
🧹 Delete Old Indices (May 0X, 2025)
DELETE /*2025.05.0*?v
Description: Deletes indices from the beginning of May 2025 (0X). Always double-check before running this to avoid accidental data loss. Use with caution.
—
⚠️ Reminder: Always verify patterns before using `DELETE` operations.

(⚡ Don’t forget to reload sysctl or reboot the server to apply permanent changes.)
🎯 Final Notes
Always monitor cluster health after major changes.
Regularly check shard allocation and reroute failures if necessary.
Keep an eye on node disk usage to avoid out-of-space errors.
Have fun and keep our ELK cluster happy! 🦌 📖 🐋