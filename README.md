usb-lock
========

A very simple bash script to unlock or lock the screen if a specific USB flash drive is plugged or unplugged.

##How to
The current version of script requires manual (but easy) configuration before usage. I'll try to extend the features in the next versions to make the initial configuration easier and without having to edit any files.

In the descriptions below, we first collect the required information for out udev file (idVendor, idProduct and Serial) and then copy the files to their correct locations.

Let's get started!

__1)__ Clone the repository or download it as [zip](https://github.com/aminbandali/usb-lock/archive/master.zip) and extract it.

__2)__ Connect your desired USB flash drive to your system. In the following command, replace _Kingston_ with the name of the vendor of your flash disk then open a terminal and execute it:  
    `lsusb | grep "Kingston"`

The output should have a syntax similar to this:  
    `Bus 001 Device 005: ID 0951:1643 Kingston Technology DataTraveler G3 4GB`

Now we have idVendor and idProduct. In this example, `idVendor` is `0951` and `idProduct` is `1643`.  
We also have the `Bus` and `Device`, which are respectly `001` and `005`. We are going to use them in the next step.

__3)__ Now we need to get the unique serial number of our flash drive, so the system will only be unlocked with our flash drive, not any others.
In the following command, replace `001` and `005` with the `Bus` and `Device` that you got from the output of the previous command. Then execute it:

    `lsusb -v -s 001:005 | awk -F " " '($1 == "iSerial") {print $3}' | grep -v ":" | grep .`  
The output is the serial number of your flash drive. For example `001373987CF5BA80D6210131`.

__4)__ Now we have all of the information that we need. Browse to the directory of the repository that you downloaded in the first step. Open this file: `91-usbkey.rules`
Initially, it looks like this:

    
    KERNEL=="sd?1", ATTRS{idVendor}=="IDVENDORHERE", ATTRS{idProduct}=="IDPRODUCT", ATTRS{serial}=="SERIALHERE", RUN+="/usr/local/bin/onusbplug.sh"  
    ACTION=="remove", ENV{ID_SERIAL_SHORT}=="SERIALHERE", RUN+="/usr/local/bin/onusbunplug.sh"
    

We will modify it in the next step.

__5)__ In the `91-usbkey.rules` file, replace `IDVENDORHERE` with the idVendor that you got from the second step. Mine was `0951`.  
Then replace `IDPRODUCTHERE` with the one you got from the second step.  
Replace the two `SERIALHERE`s with the serial number of your flash drive.

After doing the replacements, this is the new contents of my file:

    
    KERNEL=="sd?1", ATTRS{idVendor}=="0951", ATTRS{idProduct}=="1643", ATTRS{serial}=="001373987CF5BA80D6210131", RUN+="/usr/local/bin/onusbplug.sh"
    ACTION=="remove", ENV{ID_SERIAL_SHORT}=="001373987CF5BA80D6210131", RUN+="/usr/local/bin/onusbunplug.sh"
    

__6)__ Now we are ready to copy the file to its original location. Copy `91-usbkey.rules` from the repository folder to `/etc/udev/rulesd.d` directory.  
_Note: You need root access to copy the file to the specified path._

__7)__ Now we have to copy the bash scripts that do the actual lock and unlock; but before that, give them the execute permission:  
    `chmod +x onusbplug.sh && chmod +x onusbunplug.sh`

After executing the above command, move the files to `/usr/local/bin` folder.  
_Note: You need root access to copy the files to the specified path._

__8)__ Restart the udev service by typing `sudo service udev restart`.

__9)__ Copy the `.lockenabled` (which is hidden) file from the repository to your home directory and give it execution permissions:
    `chmod +x .lockenabled`

__10)__ Whenever you want to enable the lock, open a terminal and type `./.lockenabled`, then unplug your flash drive.

__11)__ Enjoy!

###Questions?
If you have any questions, just leave a comment on the [blog post](http://aminbandali.com/blog/usb-lock-version-one/) and I'll try to help you.

##Bonus tips
__Bonus tip 1:__ You can monitor the system behaviour on plug and unplugging usb devices by executing `udevadm monitor --environment --udev`. Execute the command and then plug or unplug your usb device to see the logs.

__Bonus tip 2:__ If you need information on all usb devices connect to your system, use `lsusb -v`.

__Bonus tip 3:__ If you need even more information about your usb flash drive, you can use `ls -l /dev/disk/by-id` to get the sdX where your device is connected and then use `udevadm info --query=all --path=/sys/block/sdX`.
