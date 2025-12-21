# Xfs enforce UUID

This issue occurs when mounting an LVM snapshot of an XFS filesystem.
XFS enforces filesystem UUID uniqueness at mount time.
An LVM snapshot is a block-level copy and therefore has the same UUID as the original filesystem.
When both the original XFS volume and its snapshot exist on the same system, XFS detects a duplicate UUID.
To prevent corruption, the kernel refuses to mount the snapshot.
This results in a generic “wrong fs type / bad superblock” mount error.
Ext4 does not enforce UUID uniqueness and does not have this limitation.
XFS snapshots must be mounted read-only to avoid filesystem damage.
The `nouuid` mount option tells XFS to ignore the duplicate UUID.
Correct mounting uses `mount -o ro,nouuid` for XFS LVM snapshots.
ext4 does not enforce UUID uniqueness at mount time.