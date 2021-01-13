# PyDevCon

Goal of this project is to illustrate various ways of device connectivity for network automation purposes. It will be shown based on program that connects to multiple devices, executes set of commands and records output of those commands.

Device connectivity and command configuation:

```python
tasks = [
    ("192.168.9.101", "user", "password", ("vmstat", "ip a s")),
    ("192.168.9.102", "user", "password", ("vmstat", "ip a s")),
    ("192.168.9.103", "user", "password", ("vmstat", "ip a s")),
]
```
