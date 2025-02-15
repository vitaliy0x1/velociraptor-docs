---
title: Linux.Ssh.AuthorizedKeys
hidden: true
tags: [Client Artifact]
---

Find and parse ssh authorized keys files.

```yaml
name: Linux.Ssh.AuthorizedKeys
description: Find and parse ssh authorized keys files.
parameters:
  - name: sshKeyFiles
    default: '.ssh/authorized_keys*'
    description: Glob of authorized_keys file relative to a user's home directory.

sources:
  - precondition: |
      SELECT OS From info() where OS = 'linux'

    query: |
      LET authorized_keys = SELECT * from foreach(
          row={
             SELECT Uid, User, Homedir from Artifact.Linux.Sys.Users()
          },
          query={
             SELECT FullPath, Mtime, Ctime, User, Uid
             FROM glob(root=Homedir, globs=sshKeyFiles)
          })

      SELECT * from foreach(
          row=authorized_keys,
          query={
            SELECT Uid, User, FullPath, Key, Comment, Mtime
            FROM split_records(
               filenames=FullPath, regex=" +", columns=["Type", "Key", "Comment"])
               WHERE Type =~ "ssh"
          })

```
