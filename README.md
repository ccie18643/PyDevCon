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

### Pexpect
```python
import pexpect

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

### Paramiko
```python
import paramiko

def worker(task):
    cli = paramiko.SSHClient()
    cli.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    cli.connect(hostname=task[0], username=task[1], password=task[2])
    return {cmd: str(cli.exec_command(cmd)[1].read(), encoding="utf8") for cmd in task[3]}

def poll_devices(tasks):
    return {task[0]: worker(task) for task in tasks}

def main():
    print(poll_devices(tasks))
```

### Netmiko
```python
import netmiko

def worker(task):
    cli =  netmiko.ConnectHandler(
        ip=task[0],
        username=task[1],
        password=task[2],
        device_type="linux"
    )
    return {cmd: cli.send_command(cmd) for cmd in task[3]}

def poll_devices(tasks):
    return {task[0]: worker(task) for task in tasks}

def main():
    print(poll_devices(tasks))
    
if __name__ == "__main__":
    main()
```

### AsyncSSH
```python
import asyncio
import asyncssh

async def worker(task):
    async with asyncssh.connect(
        host=task[0], username=task[1], password=task[2], known_hosts=None
    ) as cli:
        output = {cmd: (await cli.run(cmd)).stdout for cmd in task[3]}
    return task[0], output

async def poll_devices(tasks):
    results = await asyncio.gather(*[worker(task) for task in tasks])
    return {host: output for host, output in results}

def main():
    print(asyncio.run(poll_devices(tasks)))

if __name__ == "__main__":
    main()
```
