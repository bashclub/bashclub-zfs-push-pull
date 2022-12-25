# bashclub-zfs
zfs snapshot send/receive supporting pull

## Basic Usage
    bashclub-zfs remote_host:src_pool/fs dest_pool

This will create a new snapshot on the remote host (via SSH) and "send" it to
your local host where it will be received into dest_pool/fs.

I use this on my laptop to backup my home server to a removable drive. This
*pull* functionality was something I had trouble finding in the other similar
scripts out there, and was the main reason I wrote this.

## Features
* pull from a remote source to a local (or remote) destination
  * (source and destination can each be either local or remote)
* keeps its own set of snapshots
* SSH connection is made with regular user, and uses sudo for ZFS commands
* sends incremental snapshots after first run

## Features and Limitations
* always uses sudo
* always creates recursive snapshots (`zfs snapshot -r`) -  But better use 02pull to sync single Datasets
* select to send full replication stream (`zfs send -R`) or Intermediate Snapshots (zfs send -I), default without sending Full and Intermediate!
* destination should be specified as the *parent* of the desired destination
  (`zfs recv -e`)

## Install/Requirements
* There is no need to install, just clone/download where you like and run
* Created & tested with [ZFS On Linux][1] on Ubuntu 16.04
* Requires bash(1), date(1), sudo(8), and zfs(8)
* Requires ssh(1) for remote hosts
  * If your SSH environment requires a custom port or other configuration
    options, that will need to be done in your ssh_config(5) file

## Usage/Examples
    bashclub-zfs [-hvq] [-t tag] [-k keep] [-d dateopts] src dest
      use zfs send/recv to push/pull snapshots

      src          the source fs, specified as [host:]pool/path/to/fs
      dest         the destination fs parent, specified as [host:]pool/path/to/fs
                   (the final path component of src will be appended to dest)
      -p           Alternate SSH Port, default 22
      -h           help
      -v           verbose mode
      -q           quiet mode
      -t tag       tag to use for naming snapshots (default: bashclub-zfs)
      -k keep      number of snapshots to keep on src (default: 5)
      -d dateopts  options for date(1) - used to name the snapshots (default: +%F_%T)

    # Local mode: Backup tank/system to backup/tank/system
    bashclub-zfs tank/system backup/tank

    # Pull mode: Backup tank/system on tankhost to localhost
    bashclub-zfs tankhost:tank/system backup/tank

    # Push mode: Backup tank/system on localhost to backuphost
    bashclub-zfs tank/system backuphost:backup/tank

    # Double remote mode: Backup tank/system on tankhost to backuphost
    bashclub-zfs tankhost:tank/system backuphost:backup/tank

    # In this mode, your client will establish two separate SSH sessions,
    # connect them with a pipe, and pull data from one while pushing to the
    # other. No data will be stored locally.

[1]: https://github.com/zfsonlinux/zfs
