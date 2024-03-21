# Tune the xahaud node for environments with less resources

## Stop and remove xahaud database (gets recreated on service start)
```
sudo systemctl stop xahaud.service # stop the service
sudo rm -r /opt/xahaud/db
```

## Make changes to the /opt/xahaud/etc/xahaud.cfg file
```
[ledger_history]
512 ## This number should never go below 256 and must be equal to or greater than online_delete. I think this number is to low, but seems to be the consensus of the group.

[node_size]
#tiny    ## if less than 8G RAM, uncomment this and use
#small   ## if 8G RAM, uncomment this and use
#medium  ## if 16G RAM, uncomment this and use
#huge    ## if 32G RAM or above, uncomment this and use

Note: One additional setting of 'large' exists but it is not recommended to use. See https://xrpl.org/docs/infrastructure/installation/capacity-planning/

[node_db]
type=NuDB
path=/opt/xahaud/db/nudb
advisory_delete=0
online_delete=512 ## I would make this number a bit bigger, depending on the amount of storage I had.
```

## Restart xahaud service
```
sudo systemctl start xahaud.service
```

## Test Stuff
```
xahaud server_info

Note: Wait until "server_state" : "full", is shown.
```

Note: If all is working as expected, there is no need to manually delete the db going forward. What should happen is the ledger self-prunes itself as needed.
