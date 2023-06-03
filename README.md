# PoE - Heistress Remote

***This product isn't affiliated with or endorsed by Grinding Gear Games in any way.***

Remote control for the PoE Heistress tool.
This app is being developed to allow a phone to remote control the PoE Heistress tool.

*Security: I haven't been able to configure https support so all the communication is un-encrypted to/from the remote*

# Connect to the main instance
Enter the IP address in the box to the right of the contract at the top of the screen.
After the IP has been entered, click the contract and the instance will attempt to connect.
To re-enter the IP click the contract icon again and the input box will re-appear, re-enter the IP and click the contract icon to connect again.

# 

# Build instructions
I've been simply sideloding the application onto my phone via 'apk'.

```{.sh}
yarn quasar build -m android --debug
```

```{.sh}
adb.exe install -r poe-heistress-r\dist\cordova\android\apk\debug\app-debug.apk
```