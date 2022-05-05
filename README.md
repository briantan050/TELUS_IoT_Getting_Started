# Telus IoT Starter Kit Walkthrough: Part 1

This 3-part tutorial will help get you started with the TELUS LTE-M IoT Starter Kit:
* **Part 1** will give you some background on the kit and walk you through the process of getting the kit configured to send data to your own Microsoft Azure instance.
* **Part 2** will walk you through using the IoT data in a logic app with the Copernicus open access hub API. 
* **Part 3** will walk you through displaying the IoT data in a Power BI dashboard.


### Requirements for Part 1
1. [Telus IOT Starter Kit](https://www.avnet.com/shop/us/products/avnet-engineering-services/aes-bg96-iot-sk2-g-3074457345636408150?INTCMP=tbs_low-power-wide-area_button_buy-your-kit)
2. [Microsoft Azure Account](https://azure.microsoft.com/en-ca/)
3. Basic knowledge of Command-Line Interface is an asset
4. Basic knowledge of SQL is an asset

**Important note**: If you intend to complete **Part 3** of the walkthrough, it is imperative that you use the same email for both the Microsoft Power BI Account and the Microsoft Azure account for this project. Linking the data from the Azure IoT Hub to the Power BI dashboard will only work if the same email is used for both accounts. Please test to make sure that you are able to make both accounts with the same email before starting. Register with [this link](https://powerbi.microsoft.com/en-ca/) (may require a work-email to register). 

### The Kit
The Kit Consists of 3 Parts:
1. **BG96 shield** Cat.M1/NB1 & EGPRS module with added support for GPS
A 2FF SIM connector accommodates the TELUS Starter SIM that is included in the kit for connecting to the internet via TELUS’ virtual dedicated IoT network
2. **X-NUCLEO-IKS01A2** sensor board for the STM32
It is equipped with Arduino UNO R3 connector layout and is designed around the LSM6DSL 3D accelerometer and 3D gyroscope, the LSM303AGR 3D accelerometer and 3D magnetometer, the HTS221 humidity and temperature sensor and the LPS22HB pressure sensor
3. **NUCLEO L496ZG-P MCU**
The NUCLEO-L496ZG micro-controller board is fitted with an STM32L496ZG micro-controller, clocked at 80 MHz, with 1MB Flash memory, 320 KB RAM (for development flexibility), up to 115 GPIOs, an on-board ST-LINK/V2-1 debugger/programmer, and multiple expansion interfaces (USB OTG host interface, Arduino<sup>TM</sup> Uno V3 compatible expansion headers and ST Morpho headers), and is supported by comprehensive STM32 free software libraries and examples.

### MBed OS
ARM Mbed OS is a free, open-source embedded operating system designed specifically for the "things" in the Internet of Things.

It includes all the features you need to develop a connected product based on an ARM Cortex-M microcontroller, including security, connectivity, an RTOS, and drivers for sensors and I/O devices. We will be using MBed OS to compile the code for the Nucleo L496ZG Microcontroller.

# Configuring Your IoT Hardware
The BG96 and X-NUCLEO-IKS01A2 are already connected to each other in the box.  Ensure that the switch is in the SIM position. Some important parts of the board are below:

Ensure the SIM switch is in the `SIM` position, and the SIM is inserted with the notch close to the switch.

![alt text](images/sim_details.png)
*[Image used from element14 Blog](https://www.element14.com/community/groups/mbed/blog/2018/09/21/implementing-an-azure-iot-client-using-the-mbed-os-part-2)*

Also, please ensure that the Rx/Tx slide switches are set as shown (maroon switches away from the BG96 chip:

![alt text](images/board_switches.png)
*[Image used from element14 Blog](https://www.element14.com/community/groups/mbed/blog/2018/09/21/implementing-an-azure-iot-client-using-the-mbed-os-part-2)*

Connect the BG96 with sensor module to the L496 MCU so it looks like below:

![alt text](images/iot_board.png)
*[Image used from element14 Blog](https://www.element14.com/community/groups/mbed/blog/2018/09/21/implementing-an-azure-iot-client-using-the-mbed-os-part-2)*

Now your hardware is ready to be connected and programmed.

# Getting Your Software and Services Configured
### Pre-Requisites (Download and Extract/Install)
There are several tools we’ll need to use throughout this tutorial, so let’s start by installing everything we can at this point:
1. [Git 2.35.1.2](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
2. [Python 2.7.13](https://www.python.org/downloads/release/python-2713/)
3. [Python 3.10.2](https://www.python.org/downloads/release/python-3102/)
4. [GNU ARM Embedded Toolchain 10.3-2021.10](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)
5. [Azure Command-Line Tools 2.34.1](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
6. [Tera Term 4.106](https://osdn.net/projects/ttssh2/releases/)

#### Windows
Windows users will need to add **Python 2.7.13** and **Python 3.10.2** to their user or systems PATH environment variables before proceeding:

1. Right click on **My Computer** or **This PC** and select **Properties**.
![image](https://user-images.githubusercontent.com/53897474/158488241-2ec16e0a-3eab-411c-ac95-1413398c7962.png)

2. Select **Advanced system settings**.
![image](https://user-images.githubusercontent.com/53897474/158488261-8f7026ee-c8cc-4efe-b19b-28e709a23d75.png)

3. In the **Advanced** tab, select **Environment Variables...**.
![image](https://user-images.githubusercontent.com/53897474/158872785-fe5a5685-6b8b-4c28-8083-25f375733a62.png)

4. In the **System variables** section, double click on **Path** to edit path variables.
![image](https://user-images.githubusercontent.com/53897474/158488306-a20e1dc9-7df6-4bcd-b819-fcc68db5b7bf.png)

5. Add Python 2.7.13 and Python 3.10.2 as a new PATH environment variable. Click **Browse...** and navigate to the folder it was installed.
![image](https://user-images.githubusercontent.com/53897474/158488342-6d4fd7ae-4fab-4f93-ba68-e67c45ce03e9.png)

6. Select **OK**, **OK**, and **OK** to confirm and close all windows. Python 2.7.13 and Python 3.10.2 are now added into your systems PATH environment variables.

### Install PIP, the Python Package Installer
PIP is a command-line tool that installs Python packages, it is the standard for installing requirements for Python projects and we will need to use it to gather dependencies before we can compile the MBED-OS.

1. Open the command prompt. In Windows, this is done by searching **CMD**
![image](https://user-images.githubusercontent.com/53897474/158488410-f50ece73-6799-4606-aed7-914f68e35293.png)

2. From the command line, run the following command to retrieve the PIP install script:
* `curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py`
![image](https://user-images.githubusercontent.com/53897474/158873925-e0366a3d-f06e-4260-bfbb-de5605223ece.png)

3. Run the following command to retrieve and install PIP:
* `python get-pip.py`
* or `py -3 get-pip.py` (windows)
4. Verify PIP is installed correctly and ensure your Python `setuptools` package is up-to-date by running the following command:
* `python -m pip install --upgrade setuptools`
* or `py -2 -m pip install --upgrade setuptools` (windows)
* If you encounter errors with the above command, try appending `--user` and re-run
* NOTE: Do not update your Python to the latest version from the command line, as this could lead to compatibility problems with this walkthrough. 

That's all for PIP for now, we'll reference it again a bit later.

### MBED Command Line (mbed-cli)
The mbed-cli is hosted on github and built in Python, so we can download it using `git` and compile using `Python`, now that we have made sure both are installed on our computer.

From the command-line:
1. `git clone https://github.com/ARMmbed/mbed-cli.git`
2. `cd mbed-cli`
3. `python setup.py install`
* or `py -2 -m setup.py install` (windows)

Now you should be able to run the `mbed` command from your command-line, you may need to relaunch your terminal for it to work. 

### Download the Avnet Azure IoT Client
Avnet has created a client for the TELUS IoT starter kit that, with a couple of configuration tweaks, is ready to compile and load onto your IoT board.

Get the client downloaded by running the following from the command-line, this will create a folder with loads of files, so be sure to run the command in a folder that works for you:
1. `mbed import https://github.com/Avnet/azure-iot-mbed-client`
* or `py -2 -m mbed import https://github.com/Avnet/azure-iot-mbed-client` (windows)

The import will take a while, and we can’t do too much more with the client until we get Azure up and running, so let’s jump over to Azure to get things rolling on that side.

# Configuring Azure
### Setting Up Your Azure Account
We will be using Microsoft Azure to link the IoT device to the cloud. Azure is an incredibly useful cloud platform that has built-in support for IoT and allows for simple integration with several other services.

If you don’t already have an Azure account you can sign up for a free trial which comes bundled with $250 of free credits to use for 1 month:
https://azure.microsoft.com/en-ca/

Also, if you intend to complete Part 2, make sure to check that you can create a Power BI account with the same email:
https://powerbi.microsoft.com/en-ca/

### Creating Your IoT Hub
Once you have your account created you can proceed to create a new IoT Hub from your Azure dashboard. This will be our central location for all our IoT devices to connect and send whatever data we have configured them to relay, and gives us a single point to read and act on that data. Azure has security built-in, all communications between our IoT devices to Azure will be secured, and visibility to that data is also protected. 

* Click on **Create a resource**
![alt text](images/iot_hub_create.png)

* Give your IoT a unique name
* Place it in the Canada East region and make sure your Subscription is set to **Free Trial**. 
* Your new IoT Hub should look similar to this:
![alt text](images/iot_hub_config.png)

* Proceed to **Review and Create** then create your instance. This may take a couple of minutes.

Now our IoT Hub is created! As a next step, we are going to retrieve keys that we can use to securely transport and monitor the data being sent between our IoT devices and our newly created Azure IoT Hub.

* Open your newly created IoT Hub instance, then select **Shared Access Policies** from the left-hand pane which will bring up a list of pre-created policies
* Select the one labeled **iothubowner**. 
* A new right-hand pane will appear with a list of **Shared access keys**. 
* Copy the one labeled **Connection string - primary key** and store it someplace safe for later.
![alt text](images/iot_hub_connection_string.png)

The primary key we just copied can be used from the Azure command-line to monitor all traffic being sent from our IoT devices to the Hub. We will come back to the key once we have the IoT device configured, for now there’s nothing being sent to the Hub, so monitoring would be a bit boring…

### Create Your IoT Device
The next step is to create an IoT Device instance within your IoT Hub, this will be mapped directly to the physical IoT Device you are using. 
* Open your IoT Hub
* From the left-pane, select **IoT Devices**
* Then click the **Add** button to create your new device.
![alt text](images/iot_hub_new_device.png)

* Give your new device a name that is relevant to your project, this will be how you will identify the source of the data sent to your Hub. 
* Leave the other settings as-is (“Symmetric Keys” selected and “Auto-generate keys” checked). 
* Click **Save**.

* Now that your IoT device is created, click it to bring up its “Device Details” screen. 
* From this screen copy the **Connection String - primary key** and store it with the primary key you copied earlier from the IoT Hub creation step.
![alt text](images/iot_device_connection_string.png)

* This primary key will be loaded to your IoT device to secure the communications channel between it and your IoT Hub.
* At this point we have everything we need to complete the configuration of your TELUS LTE-M IoT Starter Kit, so we’ll jump back there.

### Configure Your IoT Device for Azure
Getting back to the “Download the Avnet Azure IoT Client” step from earlier on in the tutorial, hopefully it has completed importing which should have created a folder for you named “**azure-iot-mbed-client**”, within this folder there are 3 different files we need to configure. Open the following files in your text editor of choice (**Notepad** or **Notepad++** work fine too):
1. **AvnetBG96_azure_client.cpp**
2. **mbed_settings.py**
* The **azure-iot-mbed-client** folder can be found in the current directory as shown in the command prompt. These folders may be different for you:
![image](https://user-images.githubusercontent.com/53897474/158874994-e7114935-8dd8-4d97-ae36-31f7efa70de3.png)
![image](https://user-images.githubusercontent.com/53897474/158875246-14d65349-def8-40a4-bdba-c183e38965ae.png)

#### AvnetBG96_azure_client.cpp
This file handles the sensor information gathering from the IoT board sensors, crafting the sensor data into a message payload and communicating that payload to Azure. In this tutorial we’ll leave the file logic pretty much as-is, but if you feel the need to modify the function of the board, I recommend looking back to this file at a later time.

The only thing we need to configure in this file is **connectionString** (`line 81`) and the **deviceId** (`line 83`). 

1. Set **connectionString** to the **Connection String - primary key** we just copied a couple steps ago when creating the IoT device. 
2. Set the **deviceId** to the name you used for the IoT device in Azure. 
* NOTE: The deviceId is actually part of the connection string. 
* Below is a screenshot of my configured file:
![image](https://user-images.githubusercontent.com/53897474/158883663-098a1eee-3be2-4c2e-86bd-b7e43fd42084.png)

#### mbed_settings.py
In this file we need to update the **GCC_ARM_PATH** value (`line 32`) to the location where you extracted the **GNU ARM Embedded Toolchain**. 
In my case I changed the line from `/usr/local/gcc-arm-none-eabi-7-2018-q2-update/bin/` to  
`/C:/Program Files (x86)/GNU Arm Embedded Toolchain/10 2021.10/bin/`
* NOTE: Ensure the location has a `/` at each end.
![image](https://user-images.githubusercontent.com/53897474/158878203-92c825be-f4f6-4857-aca8-474a84ff598f.png)

# Compile Time!
The following steps will get your client compiled and loaded to your board:
1. Run the terminal or command-line on your Mac or Windows PC respectively
2. Change the directory to azure-iot-mbed-client (this is created in the same directory where we ran `mbed import` above) by running the following command:
* `cd azure-iot-mbed-client`
3. Install the required Python wheel package by running the command:
* `python -m pip install wheel`
* or `py -2 -m pip install wheel` (windows)
4. Install the required Python packages by running the command:
* `python -m pip install -r mbed-os/requirements.txt`
* or `py -2 -m pip install -r mbed-os/requirements.txt` (windows)
* If you encounter errors, try appending `--user` to the abve command and re-run
5. Plug a USB cable from the L496 MCU (white board) using the micro-usb cable into your computer
6. Check to see if there is a USB drive detected called NODE_L496ZG.  This means your board is connected.
7. Run the command:
* `mbed compile -m NUCLEO_L496ZG -t GCC_ARM --profile toolchain_debug.json`
* or `py -2 -m mbed compile -m NUCLEO_L496ZG -t GCC_ARM --profile toolchain_debug.json` (windows)
* *You may need to prepend the command with `python -m` on Windows or use `sudo` on Mac*
8. If all goes well, you will see the mbed compiler start creating your new bin file.  When it is complete, the file can be found here, relative to the `azure-iot-mbed-client` directory you should still be in: `BUILD\NUCLEO_L496ZG\GCC_ARM-TOOLCHAIN_DEBUG`
9. Drag the created binary over to the NODE_L496ZG drive, this will load the new client software and reboot your IoT board

Once your board reboots it will immediately attempt to connect to the network, read sensor data and send that data to your IoT Hub.

### Example
Here’s an example of the payload sent from my device:
```
{
    "event": {
        "origin": "GarettsDemoDevice",
        "payload": "{\"ObjectName\":\"Avnet NUCLEO-L496ZG+BG96 Azure IoT Client\",\"ObjectType\":\"SensorData\",\"Version\":\"1.2\",\"ReportingDevice\":\"STL496ZG-BG96\",\"Latitude\":\" 0.000\",\"Longitude\":\" 0.000\",\"GPSTime\":\"     0\",\"GPSDate\":\"\",\"Temperature\":\"23.90\",\"Humidity\":\"89\",\"Pressure\":\"1011\",\"Tilt\":\"2\",\"ButtonPress\":\"0\",\"TOD\":\"Sat 2019-02-09 16:59:52 UTC\"}"
    }
}
```

The actual data fed into your Azure function will be the JSON contents of the `payload` object.

## Monitoring Data
If all goes well, your hub will start receiving the data from your board without incident. If any issues arise, or you just want to have a better idea of what is being sent to your Hub, it would be helpful to be able to see what exactly your board is doing and the raw data being sent.
  
With the IoT board connected to your computer, you are able to analyze the board status through the COM port.

#### MacOS
1. From your terminal and issue the command ls /dev/tty.*  This will show all the serial ports you have.  Look for /dev/tty.usbmodemxxxxx (on my Mac it was 14203), which will be the board
2. Issue the command screen /dev/tty.usbmodemxxxxx 115200 (where xxxxx is for your particular Mac).  This connects to your device and displays the terminal output with baud rate of 115200.

#### Windows
Tera Term enables us to monitor messages from the board through the COM port connected to the PC. Follow these steps:
  
1. Open Tera Term, select **Serial** and **OK** to connect to the board through the COM port.
![Tera_1](https://user-images.githubusercontent.com/53897474/158283699-37a1a32f-ab6d-4f29-b91c-125c9fb77e83.png)

2. In the **Setup** tab, select **Serial port...**
![Tera_2](https://user-images.githubusercontent.com/53897474/158283783-3852542f-3635-4d28-9033-71d6866454e3.png)

3. Change the **Speed** setting to **115200** and confirm by selecting **New setting**
![Tera_3](https://user-images.githubusercontent.com/53897474/158283827-c3dd35cd-0c17-4a84-a543-452b0d3c2e06.png)

If you don’t see anything in the terminal after following the above steps, press the black “RESET B2” button on the white board, this will reboot the board and should present you with a screen similar to this one in the terminal:
![Tera_4](https://user-images.githubusercontent.com/53897474/158283873-b790b8b8-91a5-4ba9-b95a-0b3d63a333a4.png)

If it is still having trouble connecting, place your board near a window to get better reception. 

Output will continue to produce as the board makes repeated network sends to Azure. You won’t, however, get to see the actual payload being sent.

### Monitoring Payloads Sent to Azure
The Azure CLI tool will let us monitor the payloads sent from the board to Azure. The following commands will let you see the payloads sent in real-time:
1. Issue the following command to log in to Azure from the command-line
* `az login`
* A browser will open, log in using your Azure credentials
2. Install the Azure IoT extension:
* `az extension add --name azure-iot`
3. Retrieve the “Connection String - primary key” that you copied earlier when you created your IoT Hub, with it, issue the following command in the command-line terminal:
* `az iot hub monitor-events --login "<your_connection_string"`

If all goes well you will start seeing JSON payloads as they are sent to the server:
![alt text](images/azure_cli_output.png)

## Done
Your board is now sending sensor data to Azure IoT Hub on a regular basis. In this tutorial, you have completed the following:
* Created an Azure IoT Hub
* Registered your IoT device on the Azure IoT Hub
* Compiled the Azure IoT MBed client and loaded it onto your IoT device
* Successfully sent data from your IoT device to Azure IoT Hub
* Monitored the contents of the incoming JSON payloads with the Azure CLI tool 

## Next steps
Now that you are receiving sensor data in the Azure IoT Hub, the next step is to do something with it! Choose what to do next:
* **Part 2** will connect the sensor data to an Azure Logic App, enabling you to perform specific actions with it. In this tutorial, we connect it with the Copernicus open data hub API to use the sensor's latitude and longitude to select a map and retrieve its URL for the user to download.
* **Part 3** will send the sensor data from the IoT Hub to Power BI through a Stream Analytics job, enabling you to display your data on a dashboard that updates automatically.

## Credits:
* GarettB's tutorial: [TELUS IOT Getting Started](https://github.com/garettB/TELUS_IoT_Getting_Started)
* Microsoft Azure's tutorial: [Visualize real-time sensor data from Azure IoT Hub using Power BI](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-live-data-visualization-in-power-bi)
* Dinusha Kumarasiri's tutorial: [End to end IoT Solution with Azure IoT Hub, Event Grid and Logic Apps](https://youtu.be/Wb_QT0qHGOo)
* Reza Vahidnia and F. John Dian's book: [Cellular Internet of Things for Practitioners](https://pressbooks.bccampus.ca/cellulariot/)
