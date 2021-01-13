# PyDevCon

Goal of this project is to illustrate various ways of device connectivity for network automation purposes. I am going to write program that connects to multiple devices, executes set of commands and records output of each command.

Base application:

```python
tasks = [
    ("192.168.9.101", "user", "password", ("vmstat", "ip a s")),
    ("192.168.9.102", "user", "password", ("vmstat", "ip a s")),
    ("192.168.9.103", "user", "password", ("vmstat", "ip a s")),
]

```
