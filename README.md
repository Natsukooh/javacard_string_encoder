# Security of IoT project


## Objective

The objective of the project is the following: we are given a javacard (a card with a chip that can run java programs) with its associated reader, and we have to make two things:
- a program running on the card, that waits for the computer to send a PIN code, and that, when the correct PIN code has been sent, generates a 512 bytes RSA key pair, and then waits for the computer to send a string, that will be encoded with the generated key
- a terminal running on the computer where the javacard reader is plugged, that will be a user-friendly command-line program to use the encrypter on the card. Any language can be used for this part


## Toolchain

writing a Javacard program and make it run on the card is not that easy.
The steps to follow to achieve it are the following:
1. Write a Java class extending the Applet class (an applet is a program that can run on a Javacard)
2. Compile this program with the help of the javacard JDK (the javacard SDK repository is included as a git submodule)
3. Transform the obtained .class file into a .cap file thanks to an existing tool included in the SDK
4. Delete the previous version of the applet that is on the card (if you've never sent the program to the card, this step is thus not required)
5. Send the program to the card

Once done with these steps, we can start communicating ith the card, sending it APDUs, which are messages exchanged between a computer and a card. The applet you've written will receive the APDU and process it the way you've coded it.


## Scripts

To use the toolchain, I've made a bunch of scripts usable for each step previously explained:

- The java applet is defined in `project/Encrypter.java`. Feel free to edit the code to see how the javacard ecosystem work.
- To compile the applet and transform the obtained .class into a .cap file, use the `scripts/compile.sh` script. It performs both these operations.
- To delete the previous version of the applet on the card, use `scripts/delete_applet.sh`.
- To send your applet on the card, use `scripts/send_applet.sh`
- Finally, to test your applet, use `scripts/test_applet.sh`. It will activate the right applet and send to the card an APDU with operation code 0x40, hich we define to be a sort of ping, that the card should answer with a certain special message, to make sure the applet is running and working just fine.

WARNING: to run these scripts, be sure to be in the project's root folder! I'm not a bash specialist, and the bash scripts will not work if run from somehere else.


## Dependencies

You must have the following programs are installed in order for your toolchain to work:
- The following `apt` packages: `sudo apt update && sudo apt install libusb-dev libusb-1.0-0-dev libccid pcscd libpcsclite1 libpcsclite-dev libpcsc-perl pcsc-tools`
- The `pcscd` service must run: use the `sudo service pcscd status` command to check if it is running, and if not, use the `sudo service pcscd start` command
- Java 7 or 8, NOT NEWER ; the javacard toolchain is sadly legacy, and a java version more recent than java 8 is not supported
- The `gpshell` program, installable in brew with `brew install kaoh/globalplatform/globalplatform`. Make sure `brew` is installed on your computer


### Environment variables

- JC_HOME_TOOLS: must be the path to the javacard 2.1.1 SDK (provided in the repository as a submodule)
- JAVA_HOME: your java (7 or 8) installation path
- PATH: add $JC_HOME_TOOLS/bin and $JAVA_HOME/bin to your $PATH
