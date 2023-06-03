# PoE - Heistress Remote

***This product isn't affiliated with or endorsed by Grinding Gear Games in any way.***

Remote control for the PoE Heistress tool.
This app is being developed to allow a phone to remote control the PoE Heistress tool.

*Security: I haven't been able to configure https support so all the communication is un-encrypted to/from the remote*

# Connect to the main instance
Enter the IP address in the box to the right of the contract at the top of the screen.
After the IP has been entered, click the contract and the instance will attempt to connect.
To re-enter the IP click the contract icon again and the input box will re-appear, re-enter the IP and click the contract icon to connect again.

# Heist Tracking
The only major difference from the main app is that the **undo** button next to the jobs which causes the next loot room click to decrement instead of increment.

# Pricer
Push the curio icon to cause the main app to take a screen shot. It takes a second to push it over to the remote.
When the image appears on the app you can 1 finger to scroll around and then use 2 finger touch to select the region with the item's name.
This selection can be a bit finicy but unlike the main app you can keep at in untill you highlight a region you are happy with.
Once you are happy with the selection 1 finger scroll back up to where the curio button was and press the wax seal.
The main app with do the pricing and the results should appear below the wax seal/curio.
*Its harder to select the correct region and you don't get as good of feedback on the name - so double check that the name the pricer spits out.*

# Build instructions
I've been simply sideloding the application onto my phone via 'apk'.

```{.sh}
yarn quasar build -m android --debug
```

```{.sh}
adb.exe install -r poe-heistress-r\dist\cordova\android\apk\debug\app-debug.apk
```