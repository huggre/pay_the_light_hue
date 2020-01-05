# Integrating physical devices with IOTA — Philips Hue edition

## The 14th part in a series of beginner tutorials on integrating physical devices with the IOTA protocol

![img](https://miro.medium.com/max/2448/1*oAvngElPe89aoL0DO9IZYQ.jpeg)

------

## Introduction

This is the 14th part in a series of beginner tutorials where we explore integrating physical devices with the IOTA protocol. In this tutorial we will be revisiting the use-case from the [first tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-83f4e00cc5bb) in this series, where we built a simple power circuit that would allow us to purchase services from a physical device using IOTA tokens. In this tutorial we will take on the same idea and apply it on top of an existing ecosystem of hardware and software, namely the *Philips Hue*.

This will be a two part tutorial where in the first tutorial we focus on integrating IOTA with the popular *Philips Hue*. In the second tutorial we will look at smart home devices in general and how we could integrate IOTA with the opensource *ZigBee* protocol.

*Note!*
*This is probably the easiest tutorial in this series to complete as it does not involve dealing any electronics or micro-controllers.*

------

## The use case

So, why would our hotel owner choose to implement his IOTA payment system on top of an existing ecosystem like the *Philips Hue* vs building it from scratch as as we did in first tutorial? Well, the simple answer is “plug and play”. Instead of having to deal with a lot of wiring and micro controllers, he can now simply go to his nearest electronics store, get all the smart devices (lamps, bulbs, plugs etc.) he needs and have his new payment system up and running in a couple of hours.

*Note!*
*The Philips Hue system also provides a lot of features that would be very hard to implement from scratch. Such as controlling light colors, light intensity etc.*

------

## What is Philips Hue?

*Philips Hue* is an ecosystem of color changing lamps, bulbs, LED stripes, switches, dimmers, motion sensors, smart power plugs etc. that can be controlled wirelessly from an app or the *Phillips Hue* API. In the center of the ecosystem is the *Phillips Hue Bridge.* The *Hue Bridge* functions as a common controller for the entire system. The *Hue Bridge* can manage up to 50 Hue devices simultaneously in a mesh type wireless network.

You should be able to get a *Philips Hue* starter kit with a Bridge and a couple of smart light bulbs for less than 100 USD at your nearest electronics store.

![img](https://miro.medium.com/max/1200/0*PiMNKJoF9fdUqMc7)

*Note!*
*When i started working on this tutorial i did not own a Philips Hue system, so i had to go out and get one. At that point i had no intention of installing it on a permanent basis in my home. However, after getting familiar with the system i was quite impressed by the elegance, simplicity and quality of both the hardware and software. Point being, if you do not already own a Hue system, i can highly recommend getting one, it’s pretty cool.*

------

## Installing Hue

First of all, before we can move on you have to install your *Philips Hue* system according to the documentation that comes with the system. It’s pretty straight forward and should not take more than a few minutes.

After you have installed the [Hue app](https://www2.meethue.com/en-hk/philips-hue-app) on your IOS or Android device, and verified that everything is working correctly, you need to get the IP address of your Hue Bridge.

To get the IP address of your bridge, open the Philips Hue app and go to: **Settings->Hue Bridges**, select the (i) icon on your bridge. You will now see some technical details related to your bridge. Make a note of the *IP-address* as we will need it later on in our Python script(s).

------

## Assigning IOTA addresses to Hue devices

Next thing we need to do is to is to assign a unique IOTA payment address to each individual Hue device. Simplest way of creating new IOTA addresses (including QR codes) is using the Trinity wallet. Make a note of each address as we will need them in our Python script(s) later on.

Next, print the QR code for each address on a piece of paper and attach it to, or place it next to its respective physical Hue device.

------

## Required Software and libraries

A cool features that comes with the *Philips Hue* system is that it has its own application programming interface (API), that allow developers like our selves to interface and build new apps on top of the system. And even better, a team of developers calling them self’s [**studio imaginare**](http://studioimaginaire.com/en) have already made a simple Python wrapper around the API called [**phue**](https://github.com/studioimaginaire/phue) that we will be using in our project.

You will find a github repository with documentation and installation instructions for the *phue* library [here](https://github.com/studioimaginaire/phue).

------

## The Code

Now that we have made all the preparations, let’s look at the Python code for this project.

The Python script we are using for this project is basically the same as we used for the [first tutorial](https://medium.com/coinmonks/integrating-physical-devices-with-iota-83f4e00cc5bb) with some minor adjustments. Notice that there are no longer any references to the Raspberry PI GPIO pins or library, which means that we now can run the Python script(s) from any computer inside the local network.

The Python script(s) are basically just checking the balance for each IOTA address (created in a previous step) every 10 sec. As new funds are being added to an address, the script simply turns ON it’s associated Hue device, using a Hue API call. As time passes, the script continually removes time from a local device balance, switching the device OFF (again with an API call) when the balance is empty.

*Note!
Notice that in the phue* **set_light()** *function, the first augment is a reference to the id of the hue device being targeted. It is not clear to me how device id’s are managed inside the Hue Bridge/API as i can not find a reference to the this id inside the Hue app, only the name. I can only assume the id must be related to the order in which each device was added to the bridge.*

*Important!*
*Before we can have our Python code interact with the Hue Bridge, we first need to peer your computer with the Bridge. You do this by pressing the large button on top of the Bridge followed by running the following Python code on your computer within a few seconds: (This only needs to be done once)*

And here is the Python script:

```python
#!/usr/bin/python

# Imports some Python Date/Time functions
import time
import datetime

# Imports the phue library
from phue import Bridge

# Create a Hue Bridge object
# Replace IP address with the IP address of your Hue Bridge 
b = Bridge('192.168.0.71')

# If the app is not registered and the button is not pressed, press the button and call connect() (this only needs to be run a single time)
#b.connect()

# Imports the PyOTA library
from iota import Iota
from iota import Address

# Function for checking address balance on the IOTA tangle. 
def checkbalance():

    print("Checking balance")
    gb_result = api.get_balances(address)
    balance = gb_result['balances']
    return (balance[0])

# URL to IOTA fullnode used when checking balance
iotaNode = "https://nodes.thetangle.org:443"

# Create an IOTA object
api = Iota(iotaNode, "")

# IOTA address to be checked for new light funds 
# IOTA addresses can be created using the IOTA Wallet
address = [Address(b'GTZUHQSPRAQCTSQBZEEMLZPQUPAA9LPLGWCKFNEVKBINXEXZRACVKKKCYPWPKH9AWLGJHPLOZZOYTALAWOVSIJIYVZ')]

# Get current address balance at startup and use as baseline for measuring new funds being added.   
currentbalance = checkbalance()
lastbalance = currentbalance

# Define some variables
lightbalance = 0
balcheckcount = 0
lightstatus = False

# Assign hue device ID
device_id = 1

# Main loop that executes every 1 second
while True:
    
    # Check for new funds and add to lightbalance when found.
    if balcheckcount == 10:
        currentbalance = checkbalance()
        if currentbalance > lastbalance:
            lightbalance = lightbalance + (currentbalance - lastbalance)
            lastbalance = currentbalance
        balcheckcount = 0

    # Manage light balance and light ON/OFF
    if lightbalance > 0:
        if lightstatus == False:
            print("light ON")
            b.set_light(device_id,'on', True) # Turn Hue light ON
            lightstatus=True
        lightbalance = lightbalance -1       
    else:
        if lightstatus == True:
            print("light OFF")
            b.set_light(device_id,'on', False) # Turn Hue light OFF
            lightstatus=False
 
    # Print remaining light balance     
    print(datetime.timedelta(seconds=lightbalance))

    # Increase balance check counter
    balcheckcount = balcheckcount +1

    # Pause for 1 sec.
    time.sleep(1)
```

You can download the script from [here](https://gist.github.com/huggre/a33df65c5461c4b8181701cdb0334057)

------

## Running the project

To run the project, you first need to save the script in the previous section as a text file on your computer.

Notice that Python program files uses the .py extension, so let’s save the file as **pay_the_light_hue_dev1.py**
*(\*_dev1.py* referring to the particular hue device id being controlled by the script)

Next, we need to make some minor adjustments to the script:

1. Replace the IP address in the **b = Bridge(‘192.168.0.71’)** statement with the IP address of your Hue Bridge
2. Replace the IOTA payment address with the address you created for this particular device id earlier in this tutorial.
3. Update the device_id variable according to the particular hue device you are targeting (unless the id=1, then no need to change)

To execute the script, simply start a new terminal window, navigate to the folder where you saved *pay_the_light_hue_dev1.py* and type:

**python pay_the_light_hue_dev1.py**

You should now see the code being executed in your terminal window, displaying the current light balance for device 1, and checking the devices’s IOTA address balance for new funds every 10 seconds.

------

## Pay the light

To turn on a particular Hue device, simply take your mobile phone with the Trinity wallet, scan the associated QR code for the device and transfer some IOTA’s to its IOTA address. As soon as the transaction is confirmed by the IOTA tangle, the device should turn ON and stay on until the balance is empty, depending on the amount of IOTA’s you transferred. In my example I have set the IOTA/device time ratio to be 1 IOTA for 1 second of service.

------

## Managing multiple Hue devices

If you take a look at the Python script for this tutorial you will notice that the script itself is not prepared for managing multiple Hue devices simultaneously. I guess the appropriate way of dealing with this would be to re-write the code to include some type of device list or array that would allow us to manage multiple devices in parallel. Another alternative is of course (as i did) to simply have multiple instances of Python running at the same time. Each instance running its own script with its own device id/IOTA address.

------

## Donations

If you like this tutorial and want me to continue making others, feel free to make a small donation to the IOTA address below.

![img](https://miro.medium.com/max/400/0*WR-1l5LYO9h3ZdiZ.png)

> *GTZUHQSPRAQCTSQBZEEMLZPQUPAA9LPLGWCKFNEVKBINXEXZRACVKKKCYPWPKH9AWLGJHPLOZZOYTALAWOVSIJIYVZ*
