# PyDevCon

### Python connectivity to network devices
<br>

This project aims to illustrate various ways of device connectivity for network automation purposes. This is shown based on a program that, in parallel, connects to multiple devices, executes a set of commands, and records the output of those commands.

Device connectivity and command configuration are stored in the following format. This can be easily stored in JSON or YAML formatted files.

```python
tasks = [
    ["192.168.9.101", "user", "password", ["vmstat", "ip a s"]],
    ["192.168.9.102", "user", "password", ["vmstat", "ip a s"]],
    ...
    ["192.168.9.254", "user", "password", ["vmstat", "ip a s"]],
]
```

### Pexpect - separate thread for each connection
```python
import pexpect
import concurrent.futures

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
    return {task[0]: output}

def poll_devices(tasks):
    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
            process_pool = [executor.submit(worker, task) for task in tasks]
    results = [task.result() for task in process_pool if not task.exception() and task.result()]
    return {k: v for d in results for k, v in d.items()}

def main():
    print(poll_devices(tasks))
```

### Paramiko - separate thread for each connection
```python
import paramiko
import concurrent.futures

def worker(task):
    cli = paramiko.SSHClient()
    cli.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    cli.connect(hostname=task[0], username=task[1], password=task[2])
    return {task[0]: {cmd: str(cli.exec_command(cmd)[1].read(), encoding="utf8") for cmd in task[3]}}

def poll_devices(tasks):
    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
            process_pool = [executor.submit(worker, task) for task in tasks]
    results = [task.result() for task in process_pool if not task.exception() and task.result()]
    return {k: v for d in results for k, v in d.items()}

def main():
    print(poll_devices(tasks))
```

### Netmiko - separate thread for each connection
```python
import netmiko
import concurrent.futures

def worker(task):
    cli =  netmiko.ConnectHandler(
        ip=task[0],
        username=task[1],
        password=task[2],
        device_type="linux"
    )
    return {task[0]: {cmd: cli.send_command(cmd) for cmd in task[3]}}

def poll_devices(tasks):
    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
            process_pool = [executor.submit(worker, task) for task in tasks]
    results = [task.result() for task in process_pool if not task.exception() and task.result()]
    return {k: v for d in results for k, v in d.items()}

def main():
    print(poll_devices(tasks))
```

### AsyncSSH - batch of parallel connections handled by each of the cpu cores
```python
import asyncio
import asyncssh
import multiprocessing
import concurrent.futures

async def worker(task):
    async with asyncssh.connect(
        host=task[0], username=task[1], password=task[2], known_hosts=None
    ) as cli:
        output = {cmd: (await cli.run(cmd)).stdout for cmd in task[3]}
    return {task[0]: output}

async def poll_devices(tasks):
    results = await asyncio.gather(*[worker(task) for task in tasks])
    return {k: v for d in results for k, v in d.items()}

def start_asyncio(tasks):
    return asyncio.run(poll_devices(tasks))

def start_processes(tasks):
    cpu_count = multiprocessing.cpu_count()
    batch_size = len(tasks) // cpu_count + 1
    with concurrent.futures.ProcessPoolExecutor(max_workers=cpu_count) as executor:
        process_pool = [
            executor.submit(start_asyncio, tasks[n:n + batch_size])
            for n in range(0, len(tasks), batch_size)
        ]   
    results = [task.result() for task in process_pool if not task.exception() and task.result()]
    return {k: v for d in results for k, v in d.items()}

def main():
    print(start_processes(tasks))
```

### NetDev - batch of parallel connections handled by each of the cpu cores
```python
import asyncio
import netdev
import multiprocessing
import concurrent.futures

async def worker(task):
    async with netdev.create(
        host=task[0],
        username=task[1],
        password=task[2],
        device_type="terminal"
    ) as cli:
        output = {cmd: await cli.send_command(cmd) for cmd in task[3]}
    return {task[0]: output}

async def poll_devices(tasks):
    results = await asyncio.gather(*[worker(task) for task in tasks])
    return {k: v for d in results for k, v in d.items()}

def start_asyncio(tasks):
    return asyncio.run(poll_devices(tasks))

def start_processes(tasks):
    cpu_count = multiprocessing.cpu_count()
    batch_size = len(tasks) // cpu_count + 1
    with concurrent.futures.ProcessPoolExecutor(max_workers=cpu_count) as executor:
        process_pool = [
            executor.submit(start_asyncio, tasks[n:n + batch_size])
            for n in range(0, len(tasks), batch_size)
        ]   
    results = [task.result() for task in process_pool if not task.exception() and task.result()]
    return {k: v for d in results for k, v in d.items()}

def main():
    print(start_processes(tasks))
```
