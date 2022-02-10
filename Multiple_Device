#!/usr/bin/python3
# Make sure that the file you create is in the same directory of folder of this script 
from netmiko import ConnectHandler
import re

List_host=open('file_name.txt').readlines() # this line help us to write all host or IP @ in file.txt
for hoste in List_host:
    device = {
        "device_type": "cisco_ios",
        "host":   hoste,
        "username": "username",
        "password": "pass", #or get.pass()
        "verbose":True,
    }
    command="show ip int br"
    with ConnectHandler(**device) as net_connect:
        output = net_connect.send_command_timing(command)
        print(f"\n{output}\n")
        net_connect.disconnect()
