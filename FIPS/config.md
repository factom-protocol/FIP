| FIP   | Title         | Status | Category               | Author                                     | Created    |
| ----- | ------------- | ------ | ---------------------- | ------------------------------------------ | ---------- |
| -     | Config  | Draft  | Core       | Who Soup \<<who.soup@gmail.com>\>       | 20190520   |


# Summary

Unifiying and simplifying the way the config file (`factomd.conf`) and the command line parameters are loaded. Settings are split into groups, ie Core, P2P, Database, with individual groups for every network that override on a per-network basis. All settings available in the config are also command line parameters, with the command line parameter superceding the configuration setting. Command line parameters are named `-group.name` with the option of short names for select, high use settings. 

# Motivation

At the moment, it's very inconvenient to add new flags to the node since parsing is spread out over several files and it's not clear where or which settings are overwritten in what way. The config file is defined in `util/config.go` and parsed by `state/state.go` during `engine/NetStart.go`, which will initialize the state. Parameters are parsed by `engine/factomParams.go` in `main()` and then overwritten throughout `NetStart`. Adding a config setting with flag requires editing three different packages and four files. 

If you are running multiple nodes on the same machine, such as for development work, it's also not possible to use the same configuration for every use-case (main vs testnet) and files have to be swapped. Being able to set network-specific settings would enable one config file for multiple networks. 


# Specification

The config file and command line parameters will be evaluated before the factomd node is run

1. Parse the command line flags and check for valid names, stop execution if there are invalid names or values
2. Create a configuration object with default values
3. Check if the "config" parameter is present, otherwise use default path, and parse the file
4. Overwrite the default configuration with values from the config
5. Determine the network from the config file and command line flag
6. Overwrite the config default values with custom values from the network's group
7. Overwrite settings with the ones specified by the command line flags
8. The configuration is passed to the Factomd() call that loads the node

The configuration file consists of the following sections and settings, which are all case insensitive except for the names of the networks:
```
[app]
home = "" ; optional, determine system default if empty (ie ~/.factom/m2/)
mode = FULL ; choice of FULL|SERVER



[api]
port = 8088 ; v2 endpoint

[controlpanel]
mode = readonly ; choice of disabled|readonly|readwrite
port = 8090

[db]
type = LDB ; choice of LDB|Bolt|Map
datastorepath = "data/export/"
exportdata = false ; choice of true/false
exportpath = "database/export/"

[db.ldb]
path = "database/ldb" ; relative to home

[db.bolt]
path = "database/bolt" ; relative to home

[network]
network = "MAIN" ; choice of MAIN|TEST|LOCAL or the name of the custom network, aka "fct_community_test"
timer = 600 ; directoryblock time in seconds
port = 8108
fastboot = true ; choice of true|false
bindip = "" ; optional, a specific network interface to bind the listener to
seed = "https://raw.githubusercontent.com/FactomProject/factomproject.github.io/master/seed/mainseed.txt"
special = "" ; comma separated list of ip:port addresses
tlsenabled = false ; choice of true|false
tlsprivatekey = "" ; full path to private key file
tlspubliccert = "" ; full path to public cert
rpcuser = "" ; username for control panel & api
rpcpass = "" ; password for control panel & api
corsdomains = "" ; set cross-origin resource sharing parameter
changeackheights = 0 ; specifying when to change acks for switching leader servers
exchangeratechainid = "111111118d918a8be684e0dac725493a75862ef96d2d3f43f84b26969329bf03"
exchangeratepublickey = "daf5815c2de603dbfa3e1e64f88a5cf06083307cf40da4a9b539c41832135b4a"
privatekey = ""
publickey = ""
identitychainid = ""


[network.TEST]
port = 8109
timer = 600
; seed = "https://raw.githubusercontent.com/FactomProject/factomproject.github.io/master/seed/testseed.txt"
exchangeratepublickey = "1d75de249c2fc0384fb6701b30dc86b39dc72e5a47ba4f79ef250d39e21e7a4f"

[network.LOCAL]
port = 8110
timer = 6
; seed = "https://raw.githubusercontent.com/FactomProject/factomproject.github.io/master/seed/localseed.txt"
exchangeratepublickey = "3b6a27bcceb6a42d62a3a8d02a6f0d73653215771de243a63ac048a18b59da29" ; private key = all zero

[network.fct_community_test]
port = 8110
timer = 600
seed = "https://raw.githubusercontent.com/FactomProject/communitytestnet/master/seeds/testnetseeds.txt"
bootstrapidentity               = 8888882f5002ff95fce15d20ecb7e18ae6cc4d5849b372985d856b56e492ae0f
bootstrapkey                    = 58cfccaa48a101742845df3cecde6a9f38037030842d34d0eaa76867904705ae
exchangeratepublicKey        = 58cfccaa48a101742845df3cecde6a9f38037030842d34d0eaa76867904705ae
identitychainid = FA1E000000000000000000000000000000000000000000000000000000000000



[log]
level = error ; choice of debug, info, notice, warning, error, critical, alert, emergency and none
path = "database/Log"
console = standard ; choice of standard|debug

[walletd]



```


# Implementation

The configuration code is placed in its own "config" package under `factomd/config`. To parse command line flags, the `go-flags` package is used as it provides built-in support for grouping, short/long flags, and enum types. To parse the config file, the `go-ini` package is used to improve malleability and improved group supporting (like subgroups).

# Copyright

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
