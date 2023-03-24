<div align="center">
 
# How to use RTKLIB's str2str tool to receive corrections from NTRIP Server and send them to Septentrio receiver.

## AUTHORS
  
| Name | GitHub |
|------|--------|
| Iker Uranga | <a href="https://github.com/IkerUranga10">IkerUranga10</a> </br> |    

## MAINTAINER
  
| GitHub |
|--------|
| <a href="https://github.com/septentrio-users">septentrio-users</a> </br> |    

## DO YOU HAVE ANY QUESTIONS? CONTACT SEPTENTRIO SUPPORT TEAM

| <a href="https://web.septentrio.com/GH-SSN-support ">Septentrio Support Page</a>|
|---|

## SEPTENTRIO LINKS FOR USERS
 
| Contact                                                                          | Septentrio Home Page                                                        |
|----------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| <a href="https://web.septentrio.com/GH-SSN-contact ">Septentrio Contact Page</a> | <a href="https://web.septentrio.com/GH-SSN-home">Septentrio Home Page</a> |

## DISCLAIMER
  
This set of guidelines consist of a several practical examplse to help Septentrio Module users and developers to integrate third party GNSS corrections. The guidelines are based on a concrete setup, which you may or may not use to follow the integration guidelines.

It is desirable to mention the disclaimer about that setup and the guides in general before starting reading this guide.
  
| <a href="https://github.com/septentrio-gnss/Septentrio_AgnosticCorrectionsProgram/tree/main/Receiver%20and%20Raspberry%20Setup#disclaimer">Click here to know more about the Setup in which these guides are based and general implementation documentation disclaimer</a> |
|---|

</div>

## TABLE OF CONTENTS

<!--ts-->

* [Introduction](#introduction)
* [What is RTKLIB?](#what-is-rtklib)
    * [What is str2str?](#what-is-str2str)
    * [The -b flag of rtklib 2.4.3 branch](#the--b-flag-of-rtklib-correct-branch)
* [Repository download](#repository-download) 
    * [Changing to 2.4.3 branch](#changing-to-2-4-3-branch)
* [str2str tool compilation](#str2str-tool-compilation) 
* [Receiver configuration](#receiver-configuration) 
    * [Receiver Output Configuration](#receiver-output-configuration) 
    * [Receiver Input Configuration](#receiver-input-configuration) 
* [Running str2str](#running-str2str) 
    * [Different parts in execution command](#different-parts-in-execution-command) 
    * [Different streams](#different-streams) 
* [Receiving corrections](#receiving-corrections) 
    * [Execution command example](#execution-command-example) 
* [System Scheme](#system-scheme) 
    * [Simplified version of the system scheme](#simplified-version-of-the-system-scheme) 

<!--te-->

## INTRODUCTION

This repository consists in a how-to guide for the integration of RTKLIB's str2str Application Program (AP) inside Raspberry Pi to get corrections from NTRIP Caster/Server from the corrections service provided you selected and send these corrections to a Septentrio Receiver via Serial Port. 

This guide is based on the setup suggested in the documentation of this repository.

<div align="center">
    
| <a href="https://github.com/septentrio-gnss/Septentrio_AgnosticCorrectionsProgram#set-up-guide-to-use-third-party-osr-and-ssr-corrections-with-septentrios-receivers-for-precise-positioning"> Click here to acces to our Set Up Guide to use Third Party OSR and SSR correction services with Septentrio's Receivers for precise positioning.</a> |
|---|
    
</div>

Therefore, in this case the, implementation of str2str is oriented to this setup, but the general operation of str2str is similar for other cases and its installation on a Linux based operating system is done in the same way. The compilation of str2str is done using the **make** tool.

## WHAT IS RTKLIB?

<a href="https://rtklib.com/">RTKLIB</a> is an open-source software package that enables standard and precise positioning with GNSS (Global Navigation Satellite System). The software consists of a portable program library and several application programs (APs) that utilize the library. 

**One of these APs is str2str tool.**

The library is developed by <a href="https://github.com/tomojitakasu">tomojitakasu</a> whose logo is shown below:

<p align="center">
    <img src="doc_sources/tomojitakasu.jpeg" width="30%">

You can download and install RKTLIB with its tools by entering the following command in the terminal.

    sudo apt-get install rtklib
    
### What is str2str?

<a href="https://manpages.debian.org/unstable/rtklib/str2str.1.en.html">str2str </a> is an Application Program that receives input data from a stream and divides and outputs them to one or multiple streams. The input stream can be serial, TCP client, TCP server, NTRIP client, or file. The output stream can also be serial, TCP client, TCP server, NTRIP server, or file.

This repository provides a solution for a specific case where the input stream is a corrections provider that uses NTRIP. In this case, you should have the information (host, stream, password, etc.) of the NTRIP connection of the Correction Service Provider.

### The -b flag of rtklib 2.4.3 branch

One challenge of using RTKLIB with this specific setup is that the version installed using the **sudo apt-get install rtklib does not have the -b flag implementation**. This flag is **strongly recommended** to keep the connections between the receiver and Raspberry Pi system as simple as possible.
    
The -b flag implementation allows the system to send NMEA messages through the output channel, which is the serial port used to transmit RTCM corrections messages from Raspberry Pi to the receiver. **Without the -b flag, you will need to add an additional communication channel to transmit these NMEA messages to the corrections provider**. Therefore, using the -b flag results in a greater ease of system implementation because it allows you to run a single command with a single communication channel, which is a simple USB cable between the receiver and the Raspberry Pi, if you are using the setup recommended at the beginning of this guide. 
      
<div align="center">

 **Setup:** *Raspberry Pi 4 Model B + Mosaic-Go Evaluation Kit with mosaic-X5 module inside*    
    
| <a href="https://web.septentrio.com/GH-SSN-modules">Click here to the access to the all Septentrio GNSS Modules page.</a> |
|---|
    
| <a href="https://github.com/septentrio-gnss/Septentrio_AgnosticCorrectionsProgram#set-up-guide-to-use-third-party-osr-and-ssr-corrections-with-septentrios-receivers-for-precise-positioning"> Click here to acces to our Set Up Guide to use Third Party OSR and SSR correction services with Septentrio's Receivers for precise positioning.</a> |
|---|

    
</div>

To use the -b flag implementation, we have selected the option of directly cloning the RTKLIB library repository, switching to the branch that has the -b flag implemented, and compiling only the AP that we need, i.e., str2str. The compiled str2str executable file should then be run to get corrections. This is different from other RTKLIB tools that are installed using a terminal command.
     
## REPOSITORY DOWNLOAD

For the download, the **2.4.3 b24 version** is needed. This is because **-b flag** was added to str2str tool in this version and it is in charge of passing NMEA GGA data from the receiver to the NTRIP Server/Caster. In this case, we will use RKLIB 2.4.3 b34.

In the <a href="https://rtklib.com/">RTKLIB web page</a> we have to select 2.4.3 b34, as marked in red in the following picture:

<p align="center">
    <img src="doc_sources/rtklib_versions.png" width="100%">

<div align="center">
    
| <a href="https://github.com/tomojitakasu/RTKLIB/tree/rtklib_2.4.3">Click here to navigate to 2.4.5 branch o RTKLIB's Github page, provided by tomojitakasu.</a> |
|---|
    
</div>
        
The next step is to clone this repository.

    git clone https://github.com/tomojitakasu/RTKLIB.git

**Checking if we are really in the rtklib_2.4.3 branch is crucial for this system to work**. To check it we execute the following command in the terminal:

    cd RTKLIB
    git branch -a

<p align="center">
    <img src="doc_sources/master_branch.png" width="50%">

It can be seen marked in green in the picture above that we are still in the master branch. In this branch -b option for NMEA Data streaming does not exist, so we need to change to the correct branch.

### Changing to 2.4.3 branch
    
To change to the correct RTKLIB's branch, which already has the implementation of str2str tool, we need to run this command inside the cloned repository.
    
    git checkout rtklib_2.4.3

Should appear the following message:

<div align="center">
    
*Branch 'rtklib_2.4.3' set up to track remote branch 'rtklib_2.4.3' from 'origin'.
Switched to a new branch 'rtklib_2.4.3'*
    
</div>

And if we check the current branch again:

    git branch -a

<p align="center">
    <img src="doc_sources/2.4.3_branch.png" width="50%">

We can see that we are in the correct branch. 

## STR2STR TOOL COMPILATION

For the installation, **compilation with make command is needed** inside the specific Application Program's directory we want, which is str2str for this case. To do this:

    cd app/consapp/str2str/gcc
    make

Some warnings will appear but the program should compile correctly.

## RECEIVER CONFIGURATION

For the configuration of the receiver, the NMEA Streaming is only needed. We can do it in two different ways at Septentrio's HTTP Interface: <a href="192.168.3.1">192.168.3.1</a>.

**Note:** 

For this case we are using USB OTG1 (USB On-The-Go 1) port, if you are using other channel that supports serial protocol, please adapt it to your setup (For example, any COM or USB port). To check the setup we are using and which this guide is based on (Raspberry Pi and AsteRx SB Receiver), please go to our following guide:

<div align="center">

| <a href="https://github.com/septentrio-gnss/Septentrio_AgnosticCorrectionsProgram#set-up-guide-to-use-third-party-osr-and-ssr-corrections-with-septentrios-receivers-for-precise-positioning">Set Up Guide to use Third parties OSR corrections with Septentrio's Receiver for precise positioning</a> |
|---|

</div>

It works with any mosaic receiver, as they are able to stream NMEA GGA data and to receive RTCMv3 messages.

### Receiver Output Configuration

**1st OPTION:** Upload the configuration file contained inside config folder, called *str2str_minimum_config.txt*. Its content is the following:

    setNMEAOutput, Stream1, OTG1
    setNMEAOutput, Stream1, , GGA
    setNMEAOutput, Stream1, , , msec100

You can upload and download the receiver configuration by going to *Admin> Configurations> Receiver Configurations> Current* in Septentrio's HTTP interface. Once uploaded, it should look like in the following picture.

<p align="center">
    <img src="doc_sources/receiver_config.png" width="85%">

**2nd OPTION:** Create new stream manually. In the Septentrio's HTTP interface go to *NMEA/SBF Out> (+) New NMEA Stream> USB On-The-Go port> OTG1 USB On-The-Go port 1> Select GGA and 100ms interval*. If it is well configured, the streaming should appear like in the following picture.

<p align="center">
    <img src="doc_sources/nmea_stream.png" width="85%">

### Receiver Input Configuration

For this step we have to make sure that the receiver is well configured to receive RTCMv3 data. The receiver is able to read any kind of corrections format coming from all of channels it has available by default. To check if it is well configured, in Septentrio's HTTP Interface, go to *Corrections > Corrections Input* and the selected option for the communication channel you are using shoudl be as the following picture.

<p align="center">
    <img src="doc_sources/corrections_input.png" width="80%">

In this case we are usin OTG1, so we have to be sure that is receiving in auto OR rtcmV3 mode. It is recommended to let it in auto mode. With this mode, the receiver will know automatically which mesasage protocol and which message type is been used.

## RUNNING STR2STR

Once the str2str Application Program is compiled and receiver is correctly configured to send NMEA, we need to execute the str2str application like in the following picture.

<p align="center">
    <img src="doc_sources/command.png" width="100%">

### Different parts in execution command

We have **4 parts** in the execution command:

- **Command execution**
    - We need to execute a file directly, that is why is an Application Program and not a GUI or terminal tool. Also there is an equivalent tool in RTKLIB that is called strsrv, that is exactly the same as str2str but the difference is that **strsrv** is a GUI only for Windows.

- **Input stream config**
    - The AP **only supports one input at the same time**.
    - For this case the input stream is the NTRIP Caster/Server hosted by the company you have selected for the correction service.

- **Output stream config**
    - For this setup the output stream is going to be sent to the serial port connected to the Receiver.
    - As this is the channel in which we are going to send NMEA GGA Messages, from the Receiver to the Raspberry Pi (using str2str2), we must specify the host to be sent to. (See it in blue color ar the right side of the picture above)
        - This is **only** if we want to send NMEA, using -b flag.

- **(-b Flag) NMEA config, if needed**
    - In this case we need -b flag to send NMEA for an easier implementation.
    - Here we have to specify the **output port in which we are receiving NMEA Messages.**
        - In this case, the number 1 after -b flag indicates that the NMEA stream which is going to be sent to NTRIP Caster/Server is been received from the **first output stream**. (This is because str2str tool suports several output streams at the same time.)

### Different streams

For the **Input Stream**:

We have to specify the stream type with **ntrip://**, which is NTRIP for this case and insert our NTRIP credentials, host and stream (mountpoint), as we are receiving corrections through NTRIP.

    -in ntrip://LoginID:Password@Host/Stream

For the **Ouput stream**:

As we are passing RTCM corrections to the receiver via serial port we have to specify the stream type with **serial://**, and the serial port parameters with the selected serial port, baud rate, byte size, parity, stop bit and fctr.

    -out serial://SerialPort:BaudRate:ByteSize:Parity:StopBit:Fctr#host

For the setup we are using, would be:

    -out serial://ttyACM0:115200:8:n:1:off#host

- Note that for this part of the command, we need to add **#host** (which is the NTRIP Mountpoint) at the end because we will specify later (-b flag after output stream parameters) that we are passing NMEA messages through the output 1 stream channel, so we need to specify where (which NTRIP port) do we want to send this information.

- This is not the case, but **if we would not need to send NMEA messages**, the command would not have -b, like this:

    -out serial://ttyACM0:115200:8:n:1:off

For the **NMEA stream**, we just need to add:

    -b 1

- -b to configure that we are sending NMEA messages, and 1 to specify in which **output** stream channel will be sent. In this case is the output 1 stream channel, which is the serial port.

## RECEIVING CORRECTIONS

Once the setup is well connected, RTKLIB's installed, str2str AP compiled, the receiver is configured and str2str execution command's parameters well configured, we execute this finnal command (filled with your own parameters):

    ./str2str -in ntrip://LoginID:Password@Host/Stream -out serial://ttyACM0:115200:8:n:1:off#Host -b 1

And then the Septentrio's HTTP Interface should show that the receiver is getting corrections, as shown in the picture below.

<p align="center">
    <img src="doc_sources/receiver_status.png" width="100%">

### Execution command example

For a practical example let's say that previously we have the correct setup and the receiver is well configured.

The NTRIP parameters are:

- LoginID: Septentrio_Login
- Password: str2str_password
- Host: Septentrio_Host
- Stream: Septentrio_Stream

And the serial port parameters are:

- Port name: ttyACM0
- Baud rate: 115200
- Parity: none
- Byte size: 8
- Stop bit: no
- Fctr: off

The execution command would be:

    ./str2str -in ntrip://Septentrio_Login:str2str_password@Septentrio_Host/Septentrio_Stream -out serial://ttyACM0:115200:8:n:1:off#Septentrio_Host -b 1
    
## SYSTEM SCHEME

The physical setup used for this example is the one suggested in this guide called " <a href="https://github.com/septentrio-gnss/Septentrio_AgnosticCorrectionsProgram#set-up-guide-to-use-third-party-osr-and-ssr-corrections-with-septentrios-receivers-for-precise-positioning">Set Up Guide to use Third parties OSR corrections with Septentrio's Receiver for precise positioning</a> ". In the scheme below it can be seen that the only input from the user is the parameter configuration for the str2str Application Program in the execute command and the receiver configuration to ensure that NMEA stream is been sended. Both of these are done **inside the operating system running in the Raspberry Pi**. That is why there is a white box called "User Input" inside the green box which represents the Raspberry Pi.

<p align="center">
    <img src="doc_sources/system_scheme.png" width="100%">
    
 ### Simplified version of the system scheme
 
 <p align="center">
    <img src="doc_sources/str2str_scheme.jpg" width="100%">
