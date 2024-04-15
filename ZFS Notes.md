```shell
# View snapshots for a particular pool
zfs list -t snapshot -r mypool

# View snapshot names only for a given pool (silences the header too)
zfs list -r -t snapshot -o name -H mypool

# Rename a snapshot
zfs rename mypool/dataset_foo mypool/dataset_bar

# Look for zpools
zfs import

# Import a zpool named 'tank'
zfs import tank -f

# To see where things are mounted. In Ubuntu's case, it will be at /tank
zfs get mountpoint

# See the list of upgrades to the zpools
zpool upgrade

# Perform the upgrades to ALL zpools (YOU NEED TO KNOW WHAT THIS MEANS AND WHAT
# YOU'RE DOING HERE.)
sudo zpool upgrade -a

# See when a snapshot is created. You can get other properties this way as well.
zfs get creation -t snapshot tank/dataset
```
