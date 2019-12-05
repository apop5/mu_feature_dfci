# Testing DFCI

This describes the test structure for insuring DFCI operates properly.

1. A Host System (HOST) to run the test cases.
2. A Device Under Test (DUT) to be tested. with the new DFCI supported also running current Windows.
3. Both systems on the same network.

## Overview

The DFCI tests are a collection of Robot Framework test cases.
Each Robot Framework test case collection is contained in a directory, and, as a minimum, contains a run.robot file.

Each test case collection is run manually in a proscribed order, and its status is verified before running the next test case.
The tests must be run in order, as they alter the system state, much like the real usage of DFCI.

## Equipment needed

The following equipment is needed to run the DFCI tests:

1. A system to run the test cases running a current version of Windows.
2. A System Under Test also running current version of Windows.
3. Both systems able to communicate with each other across the same network.

Optional equipment is a mechanism to collect firmware logs from the system under test.
Included is a serial port support function that looks for an FTDI serial connection.
See the Platforms\SimpleFTDI\ folder.

## Setting up the Device Under Test (DUT)

Copy the files needed for the DUT.
There is a script to help you do this.
With a removable device at drive letter D for example, issue the command **DeviceUnderTest\CollectFilesForDut.cmd D:\DfciSetup**.
This will create a directory on the USB key named DfciSetup with the required files for setting up the remote server.
Mount the USB device on the DUT and run the **SetupDUT.cmd**.
This will download and install Python 3.7.4, robotframework, robotremoteserver, and pypiwin32.
In addition, the SetupDUT command will update the firewall for the robot framework testing, and a make a couple of configuration changes to Windows for a better test experience.

## Setting up the HOST system

The HOST system requires the following software (NOTE - There are dependencies on x86-64 versions of Windows):

1. A current version of Windows 10 x86-64.
2. The current Windows SDK, available here [Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk).
3. Python 3.7.4 x86-64 (the version tested), available here [Python 3.7.4](https://www.python.org/downloads/release/python-374/).
4. python -m pip install pip --upgrade
5. python -m pip install robotframework
6. python -m pop install edk2-pytool-library

Copy the DfciTests directory, including all of the contents of the subdirectories, onto the HOST system.

## Test Cases Collections

Table of DFCI Test case collections:

| Test Case Collection | Description of Test Case |
| ----- | ----- | ----- |
| DFCI_CertChainingTest | Verifies that a ZeroTouch enroll actually prompts for authorization to Enroll when the enroll package is not signed by the proper key.|
| DFCI_InitialState | Verifies that the firmware indicates support for DFCI and that the system is not already enrolled into DFCI. |
| DFCI_InTuneBadUpdate | Tries to apply a settings package signed with the wrong key |
| DFCI_InTunePermissions | Applies multiple sets of permissions to an InTune Enrolled system. |
| DFCI_InTuneEnroll | Applies a InTune Owner, an InTune Manager, and the appropriate permissions and settings. |
| DFCI_InTuneRollCerts | Updates the Owner and Manager certificates. This test can be run multiple times as it just swaps between two sets of certificates. |
| DFCI_InTuneSettings | Applies multipl sets of settings to a InTune Enrolled system. |
| DFCI_InTuneUnenroll | Applies an InTune Owner unenroll package, that removes both the InTune Owner and the InTune Manager, resets the Permission Database, and restores settings marked No UI to their default state. |

## Note on the firmware for testing DFCI

Most of DFCI functionality can be tested without regard of the Zero Touch certificate.
To test functionality of the Zero Touch feature, the firmware needs to be built with the ZTD_Leaf.cer file instead of the ZtdRecovery.cer file.

To do this, change your platform .fdf file from:

```c
FILE FREEFORM = PCD(gZeroTouchPkgTokenSpaceGuid.PcdZeroTouchCertificateFile) {
    SECTION RAW = ZeroTouchPkg/Certs/ZeroTouch/ZtdRecovery.cer
}
```

to:

```c
FILE FREEFORM = PCD(gZeroTouchPkgTokenSpaceGuid.PcdZeroTouchCertificateFile) {
    SECTION RAW = DfciPkg/UnitTests/DfciTests/ZTD_Leaf.cer
}
```

> **WARNING: Do not ship with the ZTD_Leaf.cer certificate in your firmware**

## Running The First Test Case

Run the first test as shown replacing 11.11.11.211 with the actual IP address of the DUT.
You should expect to see similar output with all four tests passing.

```txt
DfciTests>RunDfciTest.bat TestCases\DFCI_InitialState 11.11.11.211

DfciTests>python.exe -m robot.run -L TRACE -x DFCI_InitialState.xml -A Platforms\SimpleFTDI\Args.txt -v IP_OF_DUT:11.11.11.211 -v TEST_OUTPUT_BASE:C:\TestLogs\robot\DFCI_InitialState\logs_20191113_121224 -d C:\TestLogs\robot\DFCI_InitialState\logs_20191113_121224 TestCases\DFCI_InitialState\run.robot
==============================================================================
Run :: DFCI Initial State test - Verifies that there are no enrolled identi...
==============================================================================
Ensure Mailboxes Are Clean                                            ..
.L:\Common\MU\DfciPkg\UnitTests\DfciTests\TestCases\DFCI_InitialState\run.robot
Ensure Mailboxes Are Clean                                            | PASS |
------------------------------------------------------------------------------
Get the starting DFCI Settings                                        | PASS |
------------------------------------------------------------------------------
Obtain Target Parameters From Target                                  | PASS |
------------------------------------------------------------------------------
Process Complete Testcase List                                        ..Initializing testcases
..Running test
Process Complete Testcase List                                        | PASS |
------------------------------------------------------------------------------
Run :: DFCI Initial State test - Verifies that there are no enroll... | PASS |
4 critical tests, 4 passed, 0 failed
4 tests total, 4 passed, 0 failed
==============================================================================
Output:  C:\TestLogs\robot\DFCI_InitialState\logs_20191113_121224\output.xml
XUnit:   C:\TestLogs\robot\DFCI_InitialState\logs_20191113_121224\DFCI_InitialState.xml
Log:     C:\TestLogs\robot\DFCI_InitialState\logs_20191113_121224\log.html
Report:  C:\TestLogs\robot\DFCI_InitialState\logs_20191113_121224\report.html
```

## Standard Testing

Starting with a DUT that is not enrolled in DFCI, run the tests in the following order:

1. DFCI_InitialState
2. DFCI_InTuneEnroll
3. DFCI_InTuneRollCerts
4. DFCI_InTunePermissions
5. DFCI_InTuneSettings
6. DFCI_InTuneBadUpdate
7. DFCI_InTuneUnenroll

Steps 3 through 6 can and should be repeated in any order.

## Extended Testing

This tests also start with a DUT that is not enrolled in DFCI, and will leave the system not enrolled if it completes successfully.

- DFCI_CertChainingTest

## Recovering from errors

Code issues an present issues with DFCI that may require deleting the Idenity and Permission data bases. Using privileged access of a DUT that unlocks the varstore, you can delete the two master variables of DFCI. These variable are:

1. \_SPP
2. \_IPCVN

## USB Refresh Test

The test cases DFCI_InTuneEnroll and DFCI_InTuneUnenroll have a GenUsb.bat file.
The GenUsb.bat file will generate a .dfi file that UEFI management menu can read.

```txt
GenUsb MFGNAME PRODUCTNAME SERIALNUMBER
```

If there is a space or other special characters, add double quotes as in:

```txt
GenUsb Fabrikam "Fabrikam Spelunker Kit" "SN-47599011345"
```
