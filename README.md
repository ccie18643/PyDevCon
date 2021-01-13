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

1. Pexpect
```pyhon
import pexpect


tasks = [
    ("192.168.9.102", "herman", "Osika123", ("vmstat", "ip a s")),
    ("192.168.9.103", "herman", "Osika123", ("vmstat", "ip a s")),
    ("192.168.9.104", "herman", "Osika123", ("vmstat", "ip a s")),
]


def worker(task):
    cli = pexpect.spawn(
        f"ssh -o StrictHostKeyChecking=no -l {task[1]} {task[0]}",
        timeout=60,
        encoding="utf-8",
    )
    cli.expect("password:")
    cli.sendline(task[2])
    if cli.expect([r"\$", r"denied"]):
        return None
    output = {}
    for cmd in task[3]:
        cli.sendline(cmd)
        cli.expect(r"\$")
        output[cmd] = cli.before
    return output


def poll_devices(tasks):
    return {task[0]: worker(task) for task in tasks}


def main():
    print(poll_devices(tasks))
```

