# Preface

## Authors

- Chris Snow
- Colin Cook
- Nick Bitounis

## Book License

CC0 1.0 Universal

## About this book

A primary role of a HSM is the secure management of digital keys. This book introduces digital key
management concepts and reinforces those concepts with exercises that the reader can perform on an open
source Thales HSM Simulator.

The reader should have a basic understanding of symmetric and asymmetric cryptography. Appendix A,
Introductory Books in Cryptography provides a list of introductory books in cryptography for those
wishing to learn the basics or just wanting to refresh their knowledge in the field of cryptography.

This book is developed by the community - please send me your contributions.

 - The source of this book is available here: https://github.com/snowch/hsm-guide/blob/master/book.md
 - You can view the print version of the book here: https://gitprint.com/snowch/hsm-guide/blob/master/book.md

## Audience

Undergraduate and graduate students should find that this book supplements their studies in the theoretical
concepts of cryptography with practical applications.

Software Engineers and Architects designing and building security solutions using the Thales brand of
HSM will learn concepts and patterns of key management that can be applied to their designs

## Book Organisation

Chapter 1, Introduction describes the role fulfilled by a HSM. The Thales series of HSM's are then
introduced with a short history of their evolution. Finally, an open source Thales Simulator project is
presented, with hands on exercises for the reader to install the Simulator and then connect to it from Java,
C#, and ruby clients.

Chapter 2, HSM Local Master Keys (LMKs) covers in detail the concept of the LMKs. The knowledge
gained in this chapter is fundamental to your understanding of the Thales HSM. Almost all other chapters
depend on you understanding the material covered in this chapter. This chapter concludes with exercise
with the Thales Simulator to help instill the concepts of LMKs.

Chapter 3, Key Concepts describes general key concepts that you need to know in addition to the material
covered in the previous chapter about LMKs. The knowledge in this chapter is of vital importance when
interacting with the Thales HSM. Exercises are provided with the Thales Simulator to put the concepts
learnt in this chapter into practise.

Chapter 4, Secure Key Exchange two sites that are secured with HSMs need to have a set of keys that are
shared between the HSMs. This chapter describes how keys are created and shared. Exercises are given
to setup two demo sites with the Thales Simulator and generate and share keys between the demo sites.

Chapter 5, Dynamic Key Exchange 
TODO

# Introduction

## Overview

A primary role of a HSM is the secure management of digital keys. This document describes digital key
management concepts, and also describes some key management patterns for solving specific problems.

Other acronyms for Hardware Security Modules include:

- *HSM* :  Hardware Security Module / Host Security Module
- *TRSM* :  Tamper Resistant Security Module
- *SCD* :  Secure Cryptographic Devices

TODO: Why digital key management? Life without a HSM?
TODO: Key encrypting Key versus Data Encrypting Key
TODO: Thales Simulator project overview.

## Thales History and Versions

## Introduction to the Thales Simulator

## Thales Simulator Exercises

### Setting up the Thales Simulator

In this exercise, you will download, install and run the Thales Simulator on a Windows machine.  The purpose of this exercise is to get the Thales Simulator setup ready for use in later chapters.

#### Download and Install the Thales Simulator

 1. Download ```ThalesSim.Setup.0.9.6.x86.zip``` from: [http://thalessim.codeplex.com/releases/view/88576](http://thalessim.codeplex.com/releases/view/88576)
 2. Unzip the downloaded file and execute the file ```ThalesWinSimulatorSetup.msi```, accepting the default options.
 
#### Starting the Thales Simulator

 1. Navigate to the folder where you installed the Simulator (E.g. ```C:\Program Files (x86)\NTG\ThalesSimulator```)
 2. Execute ThalesWinSimulator.exe (if your are running Windows 7, right click the file and select Run As Administrator)
 3. Click the Start Simulator button. TODO: insert image.
 4. In the Application Events window, the simulator will inform you that it could not find a file containing the LMK keys so it will create a new set of keys for you. When the simulator creates new keys in this manner, the same keys will always be created. TODO: insert image.

#### Using the Simulator Console

In this section, we will connect to the HSM Console and run a basic command, Query Host (QH) to test connectivity to the HSM.

For a full list of console commands, refer to the *Console Reference Manual* which is available from Thales. Note that the Thales Simulator only implements a subset of the commands. A list of implemented console commands can be found here [http://thalessim.codeplex.com/wikipage?title=list%20of%20supported%20console%20commands](http://thalessim.codeplex.com/wikipage?title=list%20of%20supported%20console%20commands).

#####  Connecting with the Simulator Console

In this section, we TODO describe what we are doing here

 1. Start the simulator as described in [Starting the Thales Simulator](TODO fix link)
 2. Click Connect to console.
 3. Enter the command ```QH``` followed by ENTER. You should see something similar to this: TODO insert image

#####  Connecting with a Java Client

In this session, we connect to the HSM over TCP/IP using Java. When we connect using Java, we can send Host Commands to the HSM.

In the code example, below, we send the command Perform Diagnostics (NC), and print the response
to System.out.

For a full treatment of Host Programming the Thales HSM, refer to the Thales documentation “Host
Programmer’s Manual”. For a full list of Host Commands, refer to the Thales documentation “Host
Command Reference Manual”

```java
package demo;
import java.io.BufferedOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
public class Main {

   /**
   * Runs the Command "Perform Diagnostics (NC)" and
   * prints the response to System.out
   *
   * An example response is:
   *
   * !0000ND007B44AC1DDEE2A94B0007-E000
   */
   public static void main(String[] args) throws Exception {

      Socket socket = new Socket("127.0.0.1", 9998);

      // by default the Thales Simulator has a header of 4 bytes
      // so set these to 0000
      String command = "0000NC";

      // leave two bytes for inserting the command length
      byte [] commandBuffer = (" " + command).getBytes();

      // populate the command length
      commandBuffer[0] = (byte) (command.length() / 256);
      commandBuffer[1] = (byte) (command.length() % 256);

      // Write the command to the HSM
      OutputStream out = socket.getOutputStream();
      BufferedOutputStream bufferedOut = new BufferedOutputStream(out, 1024);
      bufferedOut.write(commandBuffer);
      bufferedOut.flush();

      // Read the response from the HSM
      InputStream in = socket.getInputStream();
      int result;
      while ((result = in.read()) != -1) {
         System.out.print((char)result);
      }
      socket.close();
   }
}
```
# HSM Local Master Keys (LMKs)

## LMK Overview

Local Master Keys are a set of DES or triple DES keys. They are stored securely in the HSM making it
very difficult for an attacker to gain access to them. LMKs are the only keys that are stored in the HSM.

LMKs are not used for encrypting data, but are instead used to encrypt and decrypt other keys as these
enter or leave the HSM. LMKs are used to ensure that even if the data traffic between the HSM and an
application is recorded, the clear values of any exchanged keys are not compromised.

LMKs come in pairs and the Thales HSM contains several LMK pairs. Different LMK pairs are used to
encrypt/decrypt different types of security keys. LMK pairs are identified by two numbers, for example
LMK pair 04-05, LMK pair 14-15, etc. See the diagram below.

![HSM Local Master Keys](/src/docbkx/images/HSM_LMK_Key_Pairs.png)

Each LMK pair is assigned a code. LMK pair 04-05 is assigned code 00, while LMK pair 14-15 is assigned
code 02. The full list of HSM key pairs are listed in Table 2.1, “LMK Key Pairs”, below. Note that HSM
key pairs do not start at 00-01, instead the numbering starts at 04-05, and runs non-contiguously to 38-39.

| Key Pair  | Code  |
|---|---|
| 04-05  | 00  |
| 06-07  | 01  |
| 14-15  | 02  |
| 16-17  | 03  |
| 18-19  | 04  |
| 20-21  | 05  |
| 22-23  | 06  |
| 24-25  | 07  |
| 26-27  | 08  |
| 28-26  | 09  |
| 30-31  | 10  |
| 32-33  | 11  |
| 34-35  | 12  |
| 36-37  | 13  |
| 38-39  | 14  |

Each HSM has a unique set of LMK pairs that can be either randomly generated or loaded from smart cards. HSM users jealously guard the LMKs because the integrity of the key management security scheme depends upon them.

## LMK Variants

Back when the HSM had only a handful of LMK pairs, more than the type of keys that had to be encrypted, a way had to be found to ensure that different key types can be used but also provide a way to identify parity errors with these key types. Variants are an easy way to pseudo-multiply your LMK pairs. (TODO validate this)

Keys are encrypted under LMK pairs using either the clear value of the LMK or a variant of the LMK. An LMK variant is created by performing a XOR operation with a value on the LMK key. For example, variant 1 of an LMK is created by XORing the LMK with the value 000000000000000000000000000000A6. The Thales HSM supports 10 variants for each LMK pair, with variant 0 being the clear LMK itself. The full list of variant calculation functions can be seen in Table 2.2, “LMK Variant Calculation Function”

![HSM Local Master Keys](/src/docbkx/images/HSM_LMK_Variants.png)

# Key Concepts

# Secure Key Exchange

# Dynamic Key Exchange

# PIN block creation (clear PIN blocks)

# PIN block encryption and security zones. ZPKs and TPKs

# PIN translation

# MACing

# CVV/CV2/iCVV

# Appendix A - Introductory Books in Cryptography
