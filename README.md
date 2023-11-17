# restore-symbol

A reverse engineering tool to restore stripped symbol table for iOS app.

## How to use

### Just restore symbol of oc method

- 1. Download source code and compile.

```

git clone git@github.com:mljxxx/restore-symbol.git
cd restore-symbol
git clone git@github.com:mljxxx/class-dump.git
make
./restore-symbol
```

- 2. Restore symbol using this command. It will output a new mach-o file with symbol.

```

./restore-symbol /pathto/origin_mach_o_file -o /pathto/mach_o_with_symbol 

```

- 3. Copy the new mach-o file (with symbol) to app bundle, replace the origin mach-o file with new mach-o file. Resign app bundle.

```

codesign -f -s "iPhone Developer: XXXXXXX" --signing-time none --entitlement ./xxxx.app.xcent ./xxxx.app

```

- 4. Install the app bundle to iOS device, and use lldb to debug the app. Maybe you can use the ```ios-deploy```, or other way you like. If you use ```ios-deploy``` , you can execute this command.

```

brew install ios-deploy
ios-deploy -d -b xxxx.app

```
- 5. Now you can use ```b -[class method]``` to set breakpoint.

### Restore symbol of oc block

- 1. Search block symbol in IDA to get json symbol file, using script([`search_oc_block/ida_search_block.py`](https://github.com/mljxxx/restore-symbol/blob/main/search_oc_block/ida_search_block.py)) .

- 2. Use command line tool(restore-symbol) to inject oc method symbols and block symbols into mach o file.

```

./restore-symbol /pathto/origin_mach_o_file -o /pathto/mach_o_with_symbol -j /pathto/block_symbol.json

```

- 3. Other steps(resign, install, debug) are samen as above.

## Command Line Usage 
```
Usage: restore-symbol -o <output-file> [-j <json-symbol-file>] <mach-o-file>

  where options are:
        -o <output-file>           New mach-o-file path
        --disable-oc-detect        Disable auto detect and add oc method into symbol table,
                                   only add symbol in json file
        --replace-restrict         New mach-o-file will replace the LC_SEGMENT(__RESTRICT,__restrict)
                                   with LC_SEGMENT(__restrict,__restrict) to close dylib inject protection
        -j <json-symbol-file>      Json file containing extra symbol info, the key is "name","address"
                                   like this:

                                        [
                                         {
                                          "name": "main",
                                          "address": "0xXXXXXX"
                                         },
                                         {
                                          "name": "-[XXXX XXXXX]",
                                          "address": "0xXXXXXX"
                                         },
                                         ....
                                        ]

```
