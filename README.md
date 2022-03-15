# Telus IoT Starter Kit Walkthrough

This tutorial will help get you started with the TELUS LTE-M IoT Starter Kit:
* **Part 1** will give you some background on the kit and walk you through the entire process of getting the kit configured to send data to your own Microsoft Azure instance.
* **Part 2** will walk you through displaying the IoT data in a Power BI dashboard.

![Power_BI_dashboard](https://user-images.githubusercontent.com/53897474/158296508-7f430d96-0576-4017-bcd5-acc24c4dd862.png)

### Requirements for this walkthrough

1. [Telus IOT Starter Kit](https://www.avnet.com/shop/us/products/avnet-engineering-services/aes-bg96-iot-sk2-g-3074457345636408150?INTCMP=tbs_low-power-wide-area_button_buy-your-kit)
2. [Microsoft Power BI Account](https://powerbi.microsoft.com/en-ca/) (This may require a work-email to register)
3. [Microsoft Azure Account](https://azure.microsoft.com/en-ca/) linked to the same email as Power BI account
4. Basic knowledge of Command-Line Interface is an asset
5. Basic knowledge of SQL is an asset
  
**Important note**: It is imperative that you use the same email for both the Microsoft Power BI Account and the Microsoft Azure account for this project. Linking the data from the Azure IoT Hub to the Power BI dashboard will only work if the same email is used for both accounts. Please test to make sure that you are able to make both accounts with the same email before starting.  

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

It includes all the features you need to develop a connected product based on an ARM Cortex-M microcontroller, including security, connectivity, an RTOS, and drivers for sensors and I/O devices.

# Part 1: Sending IoT data to Microsoft Azure IoT Hub

## Configuring Your IoT Hardware

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

## Getting Your Software and Services Configured

### Pre-Requisites (Download and Extract/Install)
There are several tools we’ll need to use throughout this tutorial, so let’s start by installing everything we can at this point:
1. [Git 2.35.1.2](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
2. [Mercurial](https://www.mercurial-scm.org/downloads)
3. [Python 2.7.13](https://www.python.org/downloads/release/python-2713/)
4. [Python 3.10.2](https://www.python.org/downloads/release/python-3102/)
5. [GNU ARM Embedded Toolchain 10.3-2021.10](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)
6. [Azure Command-Line Tools 2.34.1](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
7. [Tera Term 4.106](https://osdn.net/projects/ttssh2/releases/)

#### Windows
Windows users will need to add **Python 2.7.13** and **Python 3.10.2** to their user or systems PATH environment variables:

Right click on **My Computer** or **This PC** and select **Properties**.
![Path_1](https://user-images.githubusercontent.com/53897474/158274241-18625179-0f41-4d7f-958e-bd030eaf9dcd.png)

Select **Advanced system settings**.
![Path_2](https://user-images.githubusercontent.com/53897474/158274062-4fd63ea9-29af-40c6-8f21-1bd8dbac4d83.png)

In the **Advanced** tab, select **Environment Variables...**.
![Path_3](https://user-images.githubusercontent.com/53897474/158274339-f1d1c140-42a8-4c83-9451-9296aba4dda0.png)

In the **System variables** section, double click on **Path** to edit path variables.
![Path_4](https://user-images.githubusercontent.com/53897474/158274566-9144989b-71aa-45ec-8e45-1e717b16d865.png)

Add Python 2.7.13 and Python 3.10.2 as a new PATH environment variable. Click **Browse...** and navigate to the folder it was installed.
![Path_5](https://user-images.githubusercontent.com/53897474/158274882-77a6d052-e535-439c-8845-d50272e9bace.png)

Select **OK**, **OK**, and **OK** to confirm and close all windows. Python 2.7.13 and Python 3.10.2 are now added into your systems PATH environment variables.

### Install PIP, the Python Package Installer

PIP is a command-line tool that installs Python packages, it is the standard for installing requirements for Python projects and we will need to use it to gather dependencies before we can compile the MBED-OS.

1. Open the command prompt. In Windows, this is done by searching **CMD**
![image](https://user-images.githubusercontent.com/53897474/158477665-4b61f797-d319-4fe7-84b8-f4f0755ba27c.png)

2. From the command line, run the following command to retrieve the PIP install script:
* `curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py`
![image](https://user-images.githubusercontent.com/53897474/158477835-e0e38696-d232-4e29-abcb-95fc21d1e2f6.png)

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

## Configuring Azure

### Setting Up Your Azure Account

Microsoft’s Azure is incredibly useful cloud platform that has built-in support for IoT and allows for simple integration with several other services. If you don’t already have an Azure account you can sign up for a free trial which comes bundled with $250 of free credits:
https://azure.microsoft.com/en-ca/

Also make sure to check that you can create a Power BI account with the same email. Power BI 
https://powerbi.microsoft.com/en-ca/

### Creating Your IoT Hub

Once you have your account created you can proceed to create a new IoT Hub from your Azure dashboard
* Click on **Create a resource**

![alt text](images/iot_hub_create.png)

* Give your IoT a unique name
* Place it in the Canada East region and make sure your Subscription is set to **Free Trial**. 
* Your new IoT Hub should look similar to this:

![alt text](images/iot_hub_config.png)

* Proceed to **Review and Create** then create your instance. This may take a couple of minutes.

Now our IoT Hub is created! This will be our central location for all our IoT devices to connect and send whatever data we have configured them to relay, and gives us a single point to read and action on that data. Azure has security built-in, all communications between our IoT devices to Azure will be secured and visibility to that data is also protected. As a next step we are going to retrieve keys that we can use to securely transport and monitor the data being sent between our IoT devices and our newly created Azure IoT Hub.

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

Getting back to the “Download the Avnet Azure IoT Client” step from earlier on in the tutorial, hopefully it has completed importing which should have created a folder for you named “**azure-iot-mbed-client**”, within this folder there are 3 different files we need to configure. Open the following files in your text editor of choice, the screenshots from below are from [Atom](https://atom.io/), but you could also use Notepad or Notepad++:
1. **AvnetBG96_azure_client.cpp**
2. **mbed_settings.py**
* The **azure-iot-mbed-client** folder can be found in the current directory as shown in the command prompt:
![image](https://user-images.githubusercontent.com/53897474/158479826-b4ad530b-1675-49a1-acde-32d9b9e0cce3.png)
![image](https://user-images.githubusercontent.com/53897474/158480041-b04096db-93ca-461b-972b-d230f304cbc3.png)

#### AvnetBG96_azure_client.cpp

This file handles the sensor information gathering from the IoT board sensors, crafting the sensor data into a message payload and communicating that payload to Azure. In this tutorial we’ll leave the file logic pretty much as-is, but if you feel the need to modify the function of the board, I recommend looking back to this file at a later time.

The only thing we need to configure in this file is the name of the IoT device (`deviceId`, line 83) and setting the connection string (`connectionString`, line 81). Set the device ID to the name you used for the IoT device in Azure, and set the connection string to the “Connection String - primary key” we just copied a couple steps ago when creating the IoT device. One thing to note, the device ID is actually part of the connection string. Below is a screenshot of my configured file:

![alt text](images/avnetbg96_azure_client_config.png)

#### mbed_settings.py

In this file we need to update the `GCC_ARM_PATH` value to the location where you extracted the **GNU ARM Embedded Toolchain**. 
In my case I changed the line from `/usr/local/gcc-arm-none-eabi-7-2018-q2-update/bin/` to:
`/Users/garett/Documents/dev/telus/iot_hack/gcc-arm-none-eabi-8-2018-q4-major/bin/`

![alt text](images/mbed_settings.py_config.png)

NOTE: Ensure you include the trailing slash, ‘/’ on a Mac, or compilation will not succeed!

## Compile Time!

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
8. If all goes well, you will see the mbed compiler start creating your new bin file.  When it is complete, the file can be found here, relative to the `azure-iot-mbed-client` directory you should still be in: `BUILD/NUCLEO_L496ZG/GCC_ARM/azure-iot-mbed-client.bin`
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

### Monitoring Data

If all goes well, your hub will start receiving the data from your board without incident. If any issues arise, or you just want to have a better idea of what is being sent to your Hub, it would be helpful to be able to see what exactly your board is doing and the raw data being sent.

### See Your Board Status

With the IoT board connected to your computer, you are able to analyze the board status through the COM port.

#### MacOS
1. From your terminal and issue the command ls /dev/tty.*  This will show all the serial ports you have.  Look for /dev/tty.usbmodemxxxxx (on my Mac it was 14203), which will be the board
2. Issue the command screen /dev/tty.usbmodemxxxxx 115200 (where xxxxx is for your particular Mac).  This connects to your device and displays the terminal output with baud rate of 115200.

#### Windows
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

### Done

Your board is now sending sensor data to Azure IoT Hub on a regular basis. In this tutorial, you have completed the following:
* Created an Azure IoT Hub
* Registered your IoT device on the Azure IoT Hub
* Compiled the Azure IoT MBed client and loaded it onto your IoT device
* Successfully sent data from your IoT device to Azure IoT Hub

The next section will send the sensor data from the IoT Hub to Power BI through a Stream Analytics job, enabling you to display your data on a dashboard that updates automatically.

# Part 2: Displaying IoT data in a Power BI dashboard

### Add a consumer group to your IoT hub

Consumer groups provide independent views into the event stream that enable apps and Azure services to independently consume data from the same Event Hub endpoint. In this section, you add a consumer group to your IoT hub's built-in endpoint that is used later in this tutorial to pull data from the endpoint.

To add a consumer group to your IoT hub, follow these steps:

1. In the Azure portal, open your **IoT hub**.
2. On the left pane, select **Built-in endpoints**. Enter a name for your new consumer group in the text box under Consumer groups.
![Consumer_1](https://user-images.githubusercontent.com/53897474/158298467-af1742f7-d951-4438-b566-22739015824d.png)
3. Click anywhere outside the text box to save the consumer group.

### Create a Stream Analytics job

1. In the Azure portal, select **Create a resource**. 
2. Type **Stream Analytics Job** in the search box and select it from the drop-down list. 
3. On the Stream Analytics job overview page, select **Create**
4. Enter the following information for the job.  

   **Job name**: The name of the job. The name must be globally unique.  
   
   **Resource group**: Use the same resource group that your IoT hub uses.  
   
   **Location**: Use the same location as your resource group.  
   
![Stream_1](https://user-images.githubusercontent.com/53897474/158303255-152163c5-effb-4cff-982d-11f3a63aaa68.png)

5. Select **Create**.  

### Add an input to the Stream Analytics job

1. Open the Stream Analytics job.
2. Under Job topology, select **Inputs**.
3. In the Inputs pane, select **Add stream input** and select **IoT Hub** from the drop-down list. 
4. On the new input pane, enter the following information:  

    **Input alias**: Enter a unique alias for the input.  
    
    **Select IoT Hub from your subscription**: Select this radio button.  
    
    **Subscription**: Select the Azure subscription you're using for this tutorial.  
    
    **IoT Hub**: Select the IoT Hub you're using for this tutorial.  
    
    **Endpoint**: Select Messaging.  
    
    **Shared access policy name**: Select the name of the shared access policy you want the Stream Analytics job to use for your IoT hub. For this tutorial, you can select service. The service policy is created by default on new IoT hubs and grants permission to send and receive on cloud-side endpoints exposed by the IoT hub. To learn more, see Access control and permissions.  
    
    **Shared access policy key**: This field is autofilled based on your selection for the shared access policy name.  
    
    **Consumer group**: Select the consumer group you created previously.  
    
    **Leave all other fields at their defaults.**  

![Stream_2](https://user-images.githubusercontent.com/53897474/158303457-08cadb96-4c33-4db4-b54e-3322d2b293a6.png)

5. Select **Save**.

### Add an output to the Stream Analytics job

1. Under Job topology, select **Outputs**.

2. In the Outputs pane, select **Add**, and then select **Power BI** from the drop-down list.

3. On the Power BI - New output pane, select **Authorize** and follow the prompts to sign in to your Power BI account.

4. After you've signed in to Power BI, enter the following information:

    **Output alias**: A unique alias for the output.  

    **Group workspace**: Select your target group workspace.  

    **Dataset name**: Enter a dataset name.  

    **Table name**: Enter a table name.  

    **Authentication mode**: Leave at the default.  

![Stream_3](https://user-images.githubusercontent.com/53897474/158303731-61c4720e-d6e9-4d56-b659-8e06d4058529.png)

5. Select **Save**.

### Configure the query of the Stream Analytics job

1. Under Job topology, select **Query**.

2. Replace the SQL query with the following:  
  
```
SELECT
    ObjectName,
    ObjectType,
    CAST(Version AS float) as Version,
    ReportingDevice,
    CAST(Latitude AS float) as Latitude,
    CAST(Longitude AS float) as Longitude,
    CAST(GPSTime AS float) as GPSTime,
    CAST(GPSDate AS float) as GPSDate,
    CAST(Temperature AS float) as Temperature,
    CAST(Humidity AS float) as Humidity,
    CAST(Pressure AS float) as Pressure,
    CAST(Tilt AS float) as Tilt,
    CAST(ButtonPress AS float) as ButtonPress,
    TOD,
    EventProcessedUtcTime,
    PartitionId,
    EventEnqueuedUtcTime,
    IoTHub
INTO
    [YourOutputAlias]
FROM
    [YourInputAlias]
```
  
3. Replace `[YourInputAlias]` with the input alias of the job.

4. Replace `[YourOutputAlias]` with the output alias of the job.

![Stream_4](https://user-images.githubusercontent.com/53897474/158303954-2352c9f8-b18f-47f9-be90-d8ee1ed0b5d8.png)

### Run the Stream Analytics job

1. In the Stream Analytics job, select **Overview**, then select **Start > Now > Start**. 
2. Once the job successfully starts, the job status changes from Stopped to Running.

![Stream_5](https://user-images.githubusercontent.com/53897474/158310269-4a83b495-7032-4e79-9721-79ec0e9b101a.png)

3. Start your sensor board and let it run until data packets have been sent. You can keep track of this using the "Monitoring Payloads sent to Azure" section of this walkthrough. 
4. Navigating back to the **Query** section of the Stream Analytics Job, you will be able to see the incoming packets being received. 

![Stream_6](https://user-images.githubusercontent.com/53897474/158312924-05a7fac4-01b3-4498-8850-06fe880f3e20.png)

### Create a Power BI report

1. Sign in to your Power BI account and select Power BI service from the top menu.
2. Select the workspace you used from the side menu, **My Workspace**.
4. Under the **All** tab, you should see the dataset that you specified when you created the output for the Stream Analytics job.
5. Hover over the dataset you created, select More options menu (the three dots to the right of the dataset name), and then select Create report.

![Stream_7](https://user-images.githubusercontent.com/53897474/158313254-17fe5a56-e725-48ed-a497-c70067b564d8.png)

### Configure a Power BI report to visualize the data

1. Select charts, tables and maps from the **Visualizations** menu to design your dashboard.

![Stream_8](https://user-images.githubusercontent.com/53897474/158314074-11abb2c0-3aff-4e3b-b6e0-6448a3fdadee.png)

2. As an example, the configuration for the Line Chart Visualization is as follows:
* drag **Temperature** into the **Values** section, and **EventEnqueuedUtcTime** into the **Axis** section.

![Stream_9](https://user-images.githubusercontent.com/53897474/158314679-522ea6b1-fb7b-48ed-9500-1ba79868f085.png)

### Share the report

1. Select **Save** to save the report. When prompted, enter a name for your report. When prompted for a sensitivity label, you can select **Public** and then select **Save**.

![Stream_10](https://user-images.githubusercontent.com/53897474/158463523-e7d53de7-dbd9-4955-a4f3-12656cfc6c9e.png)

2. Still on the report pane, select File > Embed report > Website or portal.

![Stream_11](https://user-images.githubusercontent.com/53897474/158464085-f278e198-fe58-436b-b813-1fbcf2e3f4e1.png)

* NOTE: If you get a notification to contact your administrator to enable embed code creation, you may need to contact them. Embed code creation must be enabled before you can complete this step.

3. You're provided the report link that you can share with anyone for report access and a code snippet that you can use to integrate the report into a blog or website. Copy the link in the Secure embed code window and then close the window.

![Stream_12](https://user-images.githubusercontent.com/53897474/158464973-db6ffdfd-3cd1-49d8-9f4e-e6671b0655b1.png)

4. Open a web browser and paste the link into the address bar.

![Stream_13](https://user-images.githubusercontent.com/53897474/158465257-fc6db837-5869-4277-80d5-534bbac2e22a.png)

### Done

Your dashboard is now displaying sensor data from your Azure IoT Hub. In this tutorial, you have completed the following:
* Created a consumer group in your Azure IoT hub.
* Created and configured an Azure Stream Analytics job to read temperature telemetry from your consumer group and sent it to Power BI.
* Configured a report for the data in Power BI and shared it to the web.

## Credits:
* GarettB's tutorial: [TELUS IOT Getting Started](https://github.com/garettB/TELUS_IoT_Getting_Started)
* Microsoft Azure's tutorial: [Visualize real-time sensor data from Azure IoT Hub using Power BI](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-live-data-visualization-in-power-bi)
* Dinusha Kumarasiri's tutorial: [End to end IoT Solution with Azure IoT Hub, Event Grid and Logic Apps](https://youtu.be/Wb_QT0qHGOo)
* Reza Vahidnia and F. John Dian's book: [Cellular Internet of Things for Practitioners](https://pressbooks.bccampus.ca/cellulariot/)
