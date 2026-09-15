# Cisco Multi-Device Automation with Netmiko

A simple Python automation script for connecting to multiple Cisco IOS devices with **Netmiko** and collecting interface status information.

The project reads a list of device IP addresses/hostnames from a text file, connects to each Cisco IOS device over SSH, executes:

```text
show ip int br
```

and prints the result to the terminal.

Repository: https://github.com/AbderrezzakYAHI/Cisco

---

## Overview

This project demonstrates a basic but useful network-automation workflow:

```text
                Device list
              file.txt / hosts
                    │
                    ▼
             Python script
          Multiple_Device.py
                    │
                    ▼
               Netmiko
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Cisco R1  Cisco R2  Cisco SW1
          │         │         │
          └─────────┼─────────┘
                    ▼
             show ip int br
                    │
                    ▼
              Terminal output
```

The repository currently contains:

```text
Cisco/
├── Multiple_Device.py
├── file.txt
└── README.md
```

The GitHub repository is public and currently contains six commits. citeturn0view0

---

# Features

- Connect to multiple Cisco IOS devices
- Use Netmiko for SSH-based network automation
- Read target devices from a text file
- Execute the same command on every device
- Display command output for each device
- Automatically open and close Netmiko connections using a context manager

The current Python script uses the Netmiko `ConnectHandler` API and the Cisco IOS device type. citeturn1view1

---

# Requirements

## Python

Python 3 is recommended.

Check your Python version:

```bash
python3 --version
```

or:

```bash
python --version
```

## Netmiko

Install Netmiko with:

```bash
pip install netmiko
```

On systems where `pip` points to Python 2 or is unavailable:

```bash
python3 -m pip install netmiko
```

Verify:

```bash
python3 -c "import netmiko; print(netmiko.__version__)"
```

---

# Installation

Clone the repository:

```bash
git clone https://github.com/AbderrezzakYAHI/Cisco.git
```

Enter the project:

```bash
cd Cisco
```

Install the dependency:

```bash
python3 -m pip install netmiko
```

---

# Device Inventory File

The repository contains a file named:

```text
file.txt
```

Its purpose is to contain the IP addresses or hostnames of the Cisco devices that should be contacted.

Example:

```text
192.168.1.10
192.168.1.11
192.168.1.12
```

One device should be placed on each line.

The current repository's `file.txt` contains a comment indicating that the file is intended for hostnames or IP addresses. citeturn1view2

## Important filename note

The current Python script opens:

```python
open('file_name.txt')
```

while the repository contains:

```text
file.txt
```

Therefore, either:

### Option A — rename the inventory file

```bash
mv file.txt file_name.txt
```

or preferably:

### Option B — update the Python script

Change:

```python
List_host = open('file_name.txt').readlines()
```

to:

```python
List_host = open('file.txt').readlines()
```

Option B keeps the repository consistent with the existing filename.

---

# Credentials

The current script contains placeholder credentials directly in the Python code:

```python
"username": "username",
"password": "pass",
```

This is acceptable for a basic lab demonstration, but **credentials should not be stored directly in source code**, especially in a public GitHub repository.

The script itself even comments that the password can alternatively be obtained interactively. citeturn1view1

Recommended approaches are:

- Environment variables
- `.env` files excluded by `.gitignore`
- Ansible Vault
- AWX credentials
- Secret managers
- Interactive password prompts

---

# How the Script Works

The main program is:

```text
Multiple_Device.py
```

It performs the following steps.

## 1. Import Netmiko

```python
from netmiko import ConnectHandler
```

Netmiko provides the SSH/network-device connection functionality.

The script also imports:

```python
import re
```

although the current implementation does not use `re`.

---

## 2. Read the device list

The script reads all lines from the inventory file:

```python
List_host = open('file_name.txt').readlines()
```

Each line represents one target device.

Conceptually:

```text
file.txt
   │
   ├── 192.168.1.10
   ├── 192.168.1.11
   └── 192.168.1.12
```

---

## 3. Iterate through devices

The script loops through the list:

```python
for hoste in List_host:
```

For each device, it creates a Netmiko connection definition.

---

## 4. Define the Cisco device

The current configuration uses:

```python
device = {
    "device_type": "cisco_ios",
    "host": hoste,
    "username": "username",
    "password": "pass",
    "verbose": True,
}
```

The important parameter is:

```python
"device_type": "cisco_ios"
```

This tells Netmiko that the target is a Cisco IOS device.

---

## 5. Execute the command

The script defines:

```python
command = "show ip int br"
```

This is the Cisco IOS command:

```text
show ip interface brief
```

It provides a concise overview of interface status and addressing.

---

## 6. Connect and execute

The connection is opened using:

```python
with ConnectHandler(**device) as net_connect:
```

Then the command is executed:

```python
output = net_connect.send_command_timing(command)
```

The output is printed:

```python
print(f"\n{output}\n")
```

Finally, the connection is explicitly disconnected:

```python
net_connect.disconnect()
```

The current source code follows this sequence for each host. citeturn1view1

---

# Running the Script

After configuring the inventory file and credentials:

```bash
python3 Multiple_Device.py
```

or:

```bash
python Multiple_Device.py
```

The script connects to each device sequentially and prints the `show ip interface brief` output.

Example:

```text
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     192.168.1.1     YES DHCP   up                    up
GigabitEthernet0/1     10.0.0.1        YES manual up                    up
GigabitEthernet0/2     unassigned      YES unset  administratively down down
```

---

# Recommended Improved Version

For a safer and more robust implementation, the script can be modified to:

- Remove newline characters from hosts
- Ask for credentials instead of storing them in code
- Handle connection failures
- Continue if one device is unavailable
- Display which device is being processed
- Remove the unused `re` import
- Use `file.txt` consistently

Example:

```python
#!/usr/bin/env python3

from getpass import getpass
from netmiko import ConnectHandler
from netmiko.exceptions import NetmikoAuthenticationException, NetmikoTimeoutException


USERNAME = input("Username: ")
PASSWORD = getpass("Password: ")

with open("file.txt", encoding="utf-8") as inventory:
    hosts = [line.strip() for line in inventory if line.strip() and not line.startswith("#")]

for host in hosts:
    print(f"\n{'=' * 60}")
    print(f"Connecting to {host}")
    print(f"{'=' * 60}")

    device = {
        "device_type": "cisco_ios",
        "host": host,
        "username": USERNAME,
        "password": PASSWORD,
    }

    try:
        with ConnectHandler(**device) as net_connect:
            output = net_connect.send_command("show ip interface brief")
            print(output)

    except NetmikoAuthenticationException:
        print(f"Authentication failed for {host}")

    except NetmikoTimeoutException:
        print(f"Connection timeout for {host}")

    except Exception as error:
        print(f"Error connecting to {host}: {error}")
```

This version is better suited for a real lab because credentials are not hard-coded and a failed device does not necessarily stop the whole process.

---

# Security

## Do not commit credentials

Avoid:

```python
"username": "admin",
"password": "MyPassword123"
```

in a public repository.

Instead:

```python
USERNAME = input("Username: ")
PASSWORD = getpass("Password: ")
```

or use a secure secret-management system.

## `.gitignore`

A useful `.gitignore` could contain:

```gitignore
__pycache__/
*.pyc
.env
*.log
.vscode/
.idea/
```

If you use a local credentials file, exclude it as well:

```gitignore
credentials.txt
secrets.yml
```

---

# Troubleshooting

## `ModuleNotFoundError: No module named 'netmiko'`

Install Netmiko:

```bash
python3 -m pip install netmiko
```

---

## `FileNotFoundError`

If you see:

```text
FileNotFoundError: file_name.txt
```

make sure the inventory filename matches the Python script.

The repository currently has:

```text
file.txt
```

while the script expects:

```text
file_name.txt
```

Change the script to:

```python
open("file.txt")
```

or rename the file.

---

## Authentication failure

Verify:

- Username
- Password
- SSH configuration
- Cisco VTY configuration
- AAA configuration
- Authentication method

Test manually:

```bash
ssh username@192.168.1.10
```

---

## Connection timeout

Check:

```bash
ping 192.168.1.10
```

Then verify SSH:

```bash
ssh username@192.168.1.10
```

Also check:

- Routing
- ACLs
- Firewall rules
- TCP/22 reachability
- Cisco SSH configuration

---

# Cisco Device Prerequisites

The target Cisco device must have SSH enabled.

A simplified lab configuration could look like:

```text
hostname R1

ip domain-name lab.local

username admin privilege 15 secret <PASSWORD>

crypto key generate rsa modulus 2048

ip ssh version 2

line vty 0 4
 login local
 transport input ssh
```

Make sure the device has a reachable IP address and that SSH access is permitted.

---

# Project Workflow

```text
                    file.txt
                       │
                       │
                       ▼
              Multiple_Device.py
                       │
                       ▼
                 Read devices
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           Cisco 1   Cisco 2   Cisco 3
             │         │         │
             ▼         ▼         ▼
        SSH / Netmiko / Cisco IOS
             │         │         │
             └─────────┼─────────┘
                       ▼
              show ip int br
                       │
                       ▼
                Terminal output
```




# Author

**Abderrezzak YAHI**

GitHub:

https://github.com/AbderrezzakYAHI

Project:

https://github.com/AbderrezzakYAHI/Cisco
