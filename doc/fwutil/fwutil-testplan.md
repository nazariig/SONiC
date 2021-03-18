# SONiC FW utility testplan

## High Level Design document

## Table of contents
- [About this manual](#about-this-manual)
- [Revision](#revision)
- [1 Introduction](#1-introduction)
    - [1.1 Approach overview](#11-approach-overview)
    - [1.2 Requirements](#12-requirements)
- [2 Testplan](#2-testplan)
    - [2.1 Overview](#21-overview)
    - [2.2 Testcases](#22-testcases)
        - [2.2.1 Show commands](#221-show-commands)
            - [2.2.1.1 Positive testcases](#2211-positive-testcases)
                - [2.2.1.1.1 Get components status](#22111-get-components-status)
        - [2.2.2 Install commands](#222-install-commands)
            - [2.2.2.1 Positive testcases](#2221-positive-testcases)
                - [2.2.2.1.1 Install FW from local path](#22211-install-fw-from-local-path)
                - [2.2.2.1.2 Install FW from URL](#22212-install-fw-from-url)
            - [2.2.2.2 Negative testcases](#2222-negative-testcases)
                - [2.2.2.2.1 Invalid component name](#22221-invalid-component-name)
                - [2.2.2.2.2 Invalid FW path/URL](#22222-invalid-fw-path/url)
        - [2.2.3 Update commands](#223-update-commands)
            - [2.2.3.1 Positive testcases](#2231-positive-testcases)
                - [2.2.3.1.1 Update platform components FW from current image](#22311-update-platform-components-fw-from-current-image)
                - [2.2.3.1.2 Update platform components FW from next image](#22312-update-platform-components-fw-from-next-image)
            - [2.2.3.2 Negative testcases](#2232-negative-testcases)
                - [2.2.3.2.1 Invalid platform components configuration](#22321-invalid-platform-components-configuration)

## About this manual

This document provides general information about FW utility testplan.

## Revision

| Rev | Date       | Author         | Description     |
|:---:|:----------:|:--------------:|:----------------|
| 0.1 | 01/10/2019 | Nazarii Hnydyn | Initial version |

# 1 Introduction

## 1.1 Approach overview

FW utility testing will be focused on the next aspects: CLI part and functional part.

CLI testing assumes verification of user input to ensure that parameters handling is valid.
Functional testing assumes verification of firmware manual installation and automatic update operations.

The implementation should be done using PyTest framework.

## 1.2 Requirements

**The next CLI commands should be tested:**
1. show: displays platform components FW status
2. install: starts manual platform component FW installation
3. update: starts automatic platform components FW installation

# 2 Testplan

## 2.1 Overview

The testplan assumes implementation both types of testcases: positive and negative.

The positive testcases will be mostly focused on functional verification:
1. Manual FW update
2. Automatic FW update

The negative testcases will be mostly focused on CLI verification:
1. Nonexistent component/firmware
2. Invalid/Malformed URL
3. Platform components configuration file parsing

FW utility testing also requires a configuration file to be created  
with all the related platform components data

Configuration file should provide the next info:
1. Available platform components
2. Pre-latest/Latest FW binary URLs with the corresponding versions

**Example:**
```yaml
x86_64-mlnx_msn3800-r0:
  bios:
    latest:
      firmware: "<firmware_url>"
      version: "<firmware_version>"
    pre-latest:
      firmware: "<firmware_url>"
      version: "<firmware_version>"
  cpld:
    latest:
      firmware: "<firmware_url>"
      version: "<firmware_version>"
    pre-latest:
      firmware: "<firmware_url>"
      version: "<firmware_version>"
```

**Note:** use PSU controller for 30 sec power cycle in case of CPLD update

## 2.2 Testcases

### 2.2.1 Show commands

#### 2.2.1.1 Positive testcases

##### 2.2.1.1.1 Get components status

**Setup:**
* N/A

**Steps:**
1. Get available platform components from the config file
2. Execute `fwutil show status`
3. Check the return code
    * zero is expected
4. Parse command output
5. Compare with the predefined data

**Teardown:**
* N/A

### 2.2.2 Install commands

#### 2.2.2.1 Positive testcases

##### 2.2.2.1.1 Install FW from local path

**Setup:**
* N/A

**Steps:**
1. Get pre-latest/latest available FW from the config file
2. Pick up arbitrary platform component
3. Copy pre-latest available FW to the DUT
4. Execute `fwutil install chassis component <component_name> fw -y <fw_path>`
5. Check the return code
    * zero is expected
6. Complete the FW update
    * use cold reboot in case of BIOS upgrade
    * use power cycle with 30 sec timeout in case of CPLD update
7. Wait for the DUT to be ready
8. Execute `fwutil show status`
9. Check the return code
    * zero is expected
10. Parse command output
11. Verify FW was updated
12. Copy latest available FW to the DUT
13. Execute `fwutil install chassis component <component_name> fw -y <fw_path>`
14. Check the return code
    * zero is expected
15. Parse command output
16. Verify output contains no errors
17. Verify syslog contains corresponding events
    * FW install start/end
18. Complete the FW update
    * use cold reboot in case of BIOS upgrade
    * use power cycle with 30 sec timeout in case of CPLD update
19. Wait for the DUT to be ready
20. Execute `fwutil show status`
21. Check the return code
    * zero is expected
22. Parse command output
23. Verify FW was updated

**Teardown:**
* N/A

##### 2.2.2.1.2 Install FW from URL

**Setup:**
* N/A

**Steps:**
1. Get pre-latest/latest available FW from the config file
2. Pick up arbitrary platform component
3. Execute `fwutil install chassis component <component_name> fw -y <fw_url>`
    * use pre-latest available FW
4. Check the return code
    * zero is expected
5. Complete the FW update
    * use cold reboot in case of BIOS upgrade
    * use power cycle with 30 sec timeout in case of CPLD update
6. Wait for the DUT to be ready
7. Execute `fwutil show status`
8. Check the return code
    * zero is expected
9. Parse command output
10. Verify FW was updated
11. Execute `fwutil install chassis component <component_name> fw -y <fw_url>`
    * use latest available FW
12. Check the return code
    * zero is expected
13. Parse command output
14. Verify output contains no errors
15. Verify syslog contains corresponding events
    * FW download start/end
    * FW install start/end
16. Complete the FW update
    * use cold reboot in case of BIOS upgrade
    * use power cycle with 30 sec timeout in case of CPLD update
17. Wait for the DUT to be ready
18. Execute `fwutil show status`
19. Check the return code
    * zero is expected
20. Parse command output
21. Verify FW was updated

**Teardown:**
* N/A

#### 2.2.2.2 Negative testcases

##### 2.2.2.2.1 Invalid component name

**Setup:**
* N/A

**Steps:**
1. Execute `fwutil install chassis component <component_name> fw -y <fw_path>`
    * use invalid component name
    * use config file as reference point

**Example:**
```bash
root@sonic:/home/admin# fwutil install chassis component BIOS1 fw -y /etc/mlnx/fw/sn3800/chassis1/bios.rom
```

2. Check the return code
    * non zero is expected
3. Parse command output
4. Expect error message

```bash
Error: Invalid value for "<component_name>": Component "BIOS1" does not exist.
```

**Teardown:**
* N/A

##### 2.2.2.2.2 Invalid FW path/URL

**Setup:**
* N/A

**Steps:**
1. Execute `fwutil install chassis component <component_name> fw -y <fw_path>`
    * use invalid FW path
    * use config file as reference point

**Example:**
```bash
root@sonic:/home/admin# fwutil install chassis component BIOS fw -y /etc/mlnx/fw/sn3800/chassis1/invalid-bios.rom
```

2. Check the return code
    * non zero is expected
3. Parse command output
4. Expect error message

```bash
Error: Invalid value for "fw_path": Path "/etc/mlnx/fw/sn3800/chassis1/invalid-bios.rom" does not exist.
```

5. Execute `fwutil install chassis component <component_name> fw -y <fw_url>`
    * use invalid FW URL
    * use config file as reference point

**Example:**
```bash
root@sonic:/home/admin# fwutil install chassis component BIOS fw -y http://mlnx/fw/sn3800/chassis1/bios.rom
```

6. Check the return code
    * non zero is expected
7. Parse command output
8. Expect error message

```bash
Error: Did not receive a response from remote machine. Aborting...
```

**Teardown:**
* N/A

### 2.2.3 Update commands

#### 2.2.3.1 Positive testcases

##### 2.2.3.1.1 Update platform components FW from current image

**Setup:**
1. Backup original `platform_components.json`

**Steps:**
1. Get pre-latest/latest available FW from the config file
2. Generate `platform_components.json` with the pre-latest available FW
3. Copy pre-latest available FW to the DUT
4. Execute `fwutil update -f -y`
5. Check the return code
    * zero is expected
6. Complete the FW update
    * use cold reboot in case of BIOS upgrade
    * use power cycle with 30 sec timeout in case of CPLD update
7. Wait for the DUT to be ready
8. Execute `fwutil show status`
9. Check the return code
    * zero is expected
10. Parse command output
11. Verify FW was updated
12. Generate `platform_components.json` with the latest available FW
13. Copy latest available FW to the DUT
14. Execute `fwutil update -y --image=current`
15. Check the return code
    * zero is expected
16. Parse command output
17. Verify update summary contains no errors
18. Verify syslog contains corresponding events
    * FW install start/end
19. Complete the FW update
    * use cold reboot in case of BIOS upgrade
    * use power cycle with 30 sec timeout in case of CPLD update
20. Wait for the DUT to be ready
21. Execute `fwutil show status`
22. Check the return code
    * zero is expected
23. Parse command output
24. Verify FW was updated

**Teardown:**
1. Restore original `platform_components.json`

##### 2.2.3.1.2 Update platform components FW from next image

**Setup:**
1. Ensure DUT has two images (current/next) installed
    * use `sonic_installer list` to get the list of available images
2. Backup original `platform_components.json` on both (current/next) images
3. Reboot to the second image
4. Get latest available FW from the config file
5. Generate `platform_components.json` with the latest available FW
6. Copy latest available FW to the DUT
7. Reboot to the first image

**Steps:**
1. Get pre-latest available FW from the config file
2. Generate `platform_components.json` with the pre-latest available FW
3. Copy pre-latest available FW to the DUT
4. Execute `fwutil update -f -y`
5. Check the return code
    * zero is expected
6. Complete the FW update
    * use cold reboot in case of BIOS upgrade
    * use power cycle with 30 sec timeout in case of CPLD update
7. Wait for the DUT to be ready
8. Execute `fwutil show status`
9. Check the return code
    * zero is expected
10. Parse command output
11. Verify FW was updated
12. Execute `sonic_installer set_next_boot <next_image>`
    * use `sonic_installer list` to get the list of available images
13. Check the return code
    * zero is expected
14. Execute `fwutil update -y --image=next`
15. Check the return code
    * zero is expected
16. Parse command output
17. Verify update summary contains no errors
18. Verify syslog contains corresponding events
    * FW install start/end
19. Complete the FW update
    * use cold reboot in case of BIOS upgrade
    * use power cycle with 30 sec timeout in case of CPLD update
20. Wait for the DUT to be ready
21. Execute `fwutil show status`
22. Check the return code
    * zero is expected
23. Parse command output
24. Verify FW was updated

**Teardown:**
1. Remove image if installation took place
    * use `sonic_installer list` to get the list of available images
2. Restore original `platform_components.json` on both (current/next) images
3. Reboot to the original image

#### 2.2.3.2 Negative testcases

##### 2.2.3.2.1 Invalid platform components configuration

**Setup:**
1. Backup original `platform_components.json`

**Steps:**
1. Generate `platform_components.json` with the invalid platform structure
    * use config file as reference point

**Example:**
```json
{
    "invalid_chassis": {
        "Chassis1": {
            "component": {
                "BIOS": {
                    "firmware": "/etc/mlnx/fw/sn3800/chassis1/bios.rom",
                    "version": "0ACLH004_02.02.007_9600",
                    "info": "Cold reboot is required"
                },
                "CPLD": {
                    "firmware": "/etc/mlnx/fw/sn3800/chassis1/cpld.rom",
                    "version": "5.3.3.1",
                    "info": "Power cycle is required"
                }
            }
        }
    }
}
```

2. Execute `fwutil update -y`
3. Check the return code
    * non zero is expected
4. Parse command output
5. Expect error message

**Example:**
```bash
Error: Failed to parse "platform_components.json": invalid platform schema: "chassis" key hasn't been found. Aborting...
```

6. Generate `platform_components.json` with the invalid chassis structure
    * use config file as reference point

**Example:**
```json
{
    "chassis": {
        "Chassis1": {
            "invalid_component": {
                "BIOS": {
                    "firmware": "/etc/mlnx/fw/sn3800/chassis1/bios.rom",
                    "version": "0ACLH004_02.02.007_9600",
                    "info": "Cold reboot is required"
                },
                "CPLD": {
                    "firmware": "/etc/mlnx/fw/sn3800/chassis1/cpld.rom",
                    "version": "5.3.3.1",
                    "info": "Power cycle is required"
                }
            }
        }
    }
}
```

7. Execute `fwutil update -y`
8. Check the return code
    * non zero is expected
9. Parse command output
10. Expect error message

**Example:**
```bash
Error: Failed to parse "platform_components.json": invalid chassis schema: "component" key hasn't been found. Aborting...
```

11. Generate `platform_components.json` with the invalid component structure
    * use config file as reference point

**Example:**
```json
{
    "chassis": {
        "Chassis1": {
            "component": {
                "BIOS": {
                    "firmware": "/etc/mlnx/fw/sn3800/chassis1/bios.rom",
                    "version": {
                        "version": "0ACLH004_02.02.007_9600"
                    },
                    "info": "Cold reboot is required"
                },
                "CPLD": {
                    "firmware": "/etc/mlnx/fw/sn3800/chassis1/cpld.rom",
                    "version": "5.3.3.1",
                    "info": "Power cycle is required"
                }
            }
        }
    }
}
```

12. Execute `fwutil update -y`
13. Check the return code
    * non zero is expected
14. Parse command output
15. Expect error message

**Example:**
```bash
Error: Failed to parse "platform_components.json": invalid component schema: string is expected: key=version. Aborting...
```

**Teardown:**
1. Restore original `platform_components.json`
