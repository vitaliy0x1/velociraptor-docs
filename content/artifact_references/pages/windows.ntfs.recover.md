---
title: Windows.NTFS.Recover
hidden: true
tags: [Client Artifact]
---

Attempt to recover deleted files.

This artifact uploads all streams from an MFTId. If the MFT entry is
not allocated there is a chance that the cluster that contain the
actual data of the file will be intact still on the disk. Therefore
this artifact can be used to attempt to recover a deleted file.

A common use is to recover deleted directory entries using the
Windows.NTFS.I30 artifact and identify MFT entries of interest. This
is artifact can be used to attempt to recover some data.


```yaml
name: Windows.NTFS.Recover
description: |
  Attempt to recover deleted files.

  This artifact uploads all streams from an MFTId. If the MFT entry is
  not allocated there is a chance that the cluster that contain the
  actual data of the file will be intact still on the disk. Therefore
  this artifact can be used to attempt to recover a deleted file.

  A common use is to recover deleted directory entries using the
  Windows.NTFS.I30 artifact and identify MFT entries of interest. This
  is artifact can be used to attempt to recover some data.

parameters:
 - name: MFTId
   default: "81978"
 - name: Drive
   default: '\\.\C:'

precondition:
  SELECT * FROM info() where OS = 'windows'

sources:
  - name: Upload
    query: |
       SELECT *, upload(accessor="mft", file=Drive + Inode,
                        name=FullPath + "/" + Inode) AS IndexUpload
       FROM foreach(
            row=parse_ntfs(device=Drive, inode=MFTId).Attributes,
            query={
              SELECT _value.Type AS Type,
                     _value.TypeId AS TypeId,
                     _value.Id AS Id,
                     _value.Inode AS Inode,
                     _value.Size AS Size,
                     _value.Name AS Name,
                     _value.FullPath AS FullPath
              FROM scope()
            })

```
