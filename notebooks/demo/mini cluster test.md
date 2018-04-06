
# mini cluster of ESP32 - test
![Broccoli](https://raw.githubusercontent.com/Wei1234c/Broccoli/master/jpgs/Broccoli_cluster_cover.gif)

ref: [Celery Canvas](http://docs.celeryproject.org/en/latest/userguide/canvas.html)

### Imports


```python
import os
import sys
import time

sys.path.append(os.path.abspath(os.path.join('..', '..', 'codes', 'broccoli', 'client')))
sys.path.append(os.path.abspath(os.path.join('..', '..', 'codes', 'broccoli', 'node')))
sys.path.append(os.path.abspath(os.path.join('..', '..', '..', 'external', 'mqtt_network')))

import client
from collections import OrderedDict

from canvas import *
import tasks
```

    My name is Client_366
    

### Start client
啟動 client 物件。  

在本機上有一個 Broker 物件，負責：
- 管理本機上的 task queue
- 接受使用者發出的運算要求，並將之排入本機上的 task queue
- 通知遠端的 workers 協助處理 tasks
- 將工作發送給 workers 做處理
- 收集 workers 傳回的運算結果
- 將運算結果整合之後，傳回給使用者

而我們透過 client 物件來與 Broker 物件溝通


```python
the_client = client.Client()
the_client.start()

while not the_client.status['Is connected']:            
    time.sleep(1)
    print('Node not ready yet.')
```

    
    Sending 281 bytes
    Message:
    OrderedDict([('command', 'set connection name'), ('correlation_id', '2018-04-05 17:48:10.213166 1'), ('kwargs', {'name': 'Client_366'}), ('message_id', '2018-04-05 17:48:10.213166 1'), ('message_type', 'command'), ('need_result', True), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    [Connected: ('123.240.210.68', 1883)]
    [Listen to messages]
    Node not ready yet.
    

### Reset workers
如果需要確保 workers 的狀態都一致，或者需要重新 depoly Python module files 到 workers 上面去，可以發送指令給遠端的 workers，要求做 reboot 回到最初的狀態。


```python
the_client.reset_workers()
time.sleep(15)  # wait until workers ready.
```

    
    Sending 236 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:10.964663 10'), ('message_id', '2018-04-05 17:48:10.964663 10'), ('message_type', 'exec'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366'), ('to_exec', 'import machine;machine.reset()')])
    
    
    Data received: 236 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:10.964663 10'), ('message_id', '2018-04-05 17:48:10.964663 10'), ('message_type', 'exec'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366'), ('to_exec', 'import machine;machine.reset()')])
    
    No module named 'machine'
    
    Data received: 263 bytes
    Message:
    OrderedDict([('command', 'set connection name'), ('correlation_id', '6515'), ('kwargs', {'name': 'NodeMCU_b4e62d890c49'}), ('message_id', '6515'), ('message_type', 'command'), ('need_result', True), ('receiver', 'Hub'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 263 bytes
    Message:
    OrderedDict([('command', 'set connection name'), ('correlation_id', '6420'), ('kwargs', {'name': 'NodeMCU_b4e62d890499'}), ('message_id', '6420'), ('message_type', 'command'), ('need_result', True), ('receiver', 'Hub'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 263 bytes
    Message:
    OrderedDict([('command', 'set connection name'), ('correlation_id', '7521'), ('kwargs', {'name': 'NodeMCU_b4e62d891371'}), ('message_id', '7521'), ('message_type', 'command'), ('need_result', True), ('receiver', 'Hub'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 263 bytes
    Message:
    OrderedDict([('command', 'set connection name'), ('correlation_id', '7677'), ('kwargs', {'name': 'NodeMCU_b4e62d890a95'}), ('message_id', '7677'), ('message_type', 'command'), ('need_result', True), ('receiver', 'Hub'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    

### Upload tasks module file
Reboot 之後的 workers 只有最基本的功能，如果我們需要 workers 執行額外的功能 (functions)，則需要先將定義這些 functions 的 module 檔案上傳給 workers 並且要求它們 import。  

例如： 我們上傳給每一個 worker 一個`tasks.py`的檔案，其中定義幾個 functions:
```
from canvas import Task

@Task
def add(x, y, op=None):
    return op(x, y) if op else x + y

@Task
def xsum(x):
    return sum(x)

@Task
def mul(x, y, op=None):
    return op(x, y) if op else x * y

@Task
def mapper(word):
    return (word, 1) if len(word) > 3 else None

```

Workers 收到這個`tasks.py`檔案之後，會做`import tasks`的動作，所以就可以呼叫 tasks.add() 這個 function。


```python
tasks_file = os.path.join('..', '..', 'codes', 'broccoli', 'client', 'tasks.py')
the_client.sync_file(tasks_file, load_as_tasks = True)
```

    
    Sending 550 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:26.045000 25'), ('kwargs', {'filename': 'tasks.py', 'file': 'from canvas import Task\n\n@Task\ndef add(x, y, op=None):\n    return op(x, y) if op else x + y\n\n@Task\ndef xsum(x):\n    return sum(x)\n\n@Task\ndef mul(x, y, op=None):\n    return op(x, y) if op else x * y\n\n@Task\ndef mapper(word):\n    return (word, 1) if len(word) > 3 else None\n', 'load_as_tasks': True}), ('message_id', '2018-04-05 17:48:26.045000 25'), ('message_type', 'file'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    

<font color='blue'>
## DEMOs

### Chains
Celery [Chains](http://docs.celeryproject.org/en/latest/userguide/canvas.html#chains) 的主要作用是把多個運算**串聯**起來，前一個運算的結果是下一個運算的參數，這樣就可以組成一個完整的運算過程，例如下例中用`chain`組成一個 ((4+4) * 8) * 10  = 640 的計算過程
```
>>> # (4 + 4) * 8 * 10
>>> res = chain(add.s(4, 4), mul.s(8), mul.s(10))
proj.tasks.add(4, 4) | proj.tasks.mul(8) | proj.tasks.mul(10)

>>> res = chain(add.s(4, 4), mul.s(8), mul.s(10))()
>>> res.get()
640
```

我們可以在 ESP32 cluster 上面也做同樣的事情：


```python
ch = chain(tasks.add.s(4, 4), tasks.mul.s(8), tasks.mul.s(10))
ch.get()
```

    
    Sending 257 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:26.144669 29'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:26.144669 29'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 550 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:26.045000 25'), ('kwargs', {'filename': 'tasks.py', 'file': 'from canvas import Task\n\n@Task\ndef add(x, y, op=None):\n    return op(x, y) if op else x + y\n\n@Task\ndef xsum(x):\n    return sum(x)\n\n@Task\ndef mul(x, y, op=None):\n    return op(x, y) if op else x * y\n\n@Task\ndef mapper(word):\n    return (word, 1) if len(word) > 3 else None\n', 'load_as_tasks': True}), ('message_id', '2018-04-05 17:48:26.045000 25'), ('message_type', 'file'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 257 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:26.144669 29'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:26.144669 29'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '14385'), ('function', 'dequeue_task'), ('message_id', '14385'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '14385'), ('message_id', '2018-04-05 17:48:27.194884 71'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:26.144612 28', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:26.144612 28', 'task_id': '2018-04-05 17:48:26.144612 28'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 485 bytes
    Message:
    OrderedDict([('correlation_id', '14385'), ('message_id', '2018-04-05 17:48:27.194884 71'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:26.144612 28', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:26.144612 28', 'task_id': '2018-04-05 17:48:26.144612 28'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '14317'), ('function', 'dequeue_task'), ('message_id', '14317'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '14317'), ('message_id', '2018-04-05 17:48:27.315021 76'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 207 bytes
    Message:
    OrderedDict([('correlation_id', '14317'), ('message_id', '2018-04-05 17:48:27.315021 76'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '14295'), ('function', 'dequeue_task'), ('message_id', '14295'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '14295'), ('message_id', '2018-04-05 17:48:27.462904 81'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 207 bytes
    Message:
    OrderedDict([('correlation_id', '14295'), ('message_id', '2018-04-05 17:48:27.462904 81'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '14424'), ('function', 'dequeue_task'), ('message_id', '14424'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '14424'), ('message_id', '2018-04-05 17:48:27.596975 87'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 207 bytes
    Message:
    OrderedDict([('correlation_id', '14424'), ('message_id', '2018-04-05 17:48:27.596975 87'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 522 bytes
    Message:
    OrderedDict([('correlation_id', '14943'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 8, 'function': 'tasks.add', 'args': [4, 4], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:26.144612 28', 'message_type': 'result', 'task_id': '2018-04-05 17:48:26.144612 28', 'correlation_id': '2018-04-05 17:48:26.144612 28', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '14943'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Sending 257 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:27.747692 96'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:27.747692 96'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 257 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:27.747692 96'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:27.747692 96'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '15483'), ('function', 'dequeue_task'), ('message_id', '15483'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '15483'), ('message_id', '2018-04-05 17:48:28.405981 130'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:27.747655 95', 'function': 'tasks.mul', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:27.747655 95', 'task_id': '2018-04-05 17:48:27.747655 95'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 486 bytes
    Message:
    OrderedDict([('correlation_id', '15483'), ('message_id', '2018-04-05 17:48:28.405981 130'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:27.747655 95', 'function': 'tasks.mul', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:27.747655 95', 'task_id': '2018-04-05 17:48:27.747655 95'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '15462'), ('function', 'dequeue_task'), ('message_id', '15462'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '15462'), ('message_id', '2018-04-05 17:48:28.516864 135'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '15462'), ('message_id', '2018-04-05 17:48:28.516864 135'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '15450'), ('function', 'dequeue_task'), ('message_id', '15450'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '15450'), ('message_id', '2018-04-05 17:48:28.673897 141'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '15450'), ('message_id', '2018-04-05 17:48:28.673897 141'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '15459'), ('function', 'dequeue_task'), ('message_id', '15459'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '15459'), ('message_id', '2018-04-05 17:48:28.805752 147'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '15459'), ('message_id', '2018-04-05 17:48:28.805752 147'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 523 bytes
    Message:
    OrderedDict([('correlation_id', '16267'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 64, 'function': 'tasks.mul', 'args': [8, 8], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:27.747655 95', 'message_type': 'result', 'task_id': '2018-04-05 17:48:27.747655 95', 'correlation_id': '2018-04-05 17:48:27.747655 95', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '16267'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Sending 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:28.955285 156'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:28.955285 156'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:28.955285 156'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:28.955285 156'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '16586'), ('function', 'dequeue_task'), ('message_id', '16586'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '16586'), ('message_id', '2018-04-05 17:48:29.607527 189'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:28.955244 155', 'function': 'tasks.mul', 'args': (10, 64), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:28.955244 155', 'task_id': '2018-04-05 17:48:28.955244 155'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 491 bytes
    Message:
    OrderedDict([('correlation_id', '16586'), ('message_id', '2018-04-05 17:48:29.607527 189'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:28.955244 155', 'function': 'tasks.mul', 'args': (10, 64), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:28.955244 155', 'task_id': '2018-04-05 17:48:28.955244 155'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '16637'), ('function', 'dequeue_task'), ('message_id', '16637'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '16637'), ('message_id', '2018-04-05 17:48:29.718520 194'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '16637'), ('message_id', '2018-04-05 17:48:29.718520 194'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '16663'), ('function', 'dequeue_task'), ('message_id', '16663'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '16663'), ('message_id', '2018-04-05 17:48:29.839288 200'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '16663'), ('message_id', '2018-04-05 17:48:29.839288 200'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '16628'), ('function', 'dequeue_task'), ('message_id', '16628'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '16628'), ('message_id', '2018-04-05 17:48:29.960812 206'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '16628'), ('message_id', '2018-04-05 17:48:29.960812 206'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '17252'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 640, 'function': 'tasks.mul', 'args': [10, 64], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:28.955244 155', 'message_type': 'result', 'task_id': '2018-04-05 17:48:28.955244 155', 'correlation_id': '2018-04-05 17:48:28.955244 155', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '17252'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    




    640



### Groups
Celery [Groups](http://docs.celeryproject.org/en/latest/userguide/canvas.html#groups) 的主要作用是把多個運算**併聯**起來，把很多同質性的運算同時發送給許多遠端的 workers 協助處理，再收集 workers 傳回來的結果彙整成為一個結果集，例如下例中用`group`同時計算 (2+2) 和 (4+4)，結果是 [4, 8]
```
>>> from celery import group
>>> from proj.tasks import add

>>> group(add.s(2, 2), add.s(4, 4))
(proj.tasks.add(2, 2), proj.tasks.add(4, 4))  

>>> g = group(add.s(2, 2), add.s(4, 4))
>>> res = g()
>>> res.get()
[4, 8]
```
我們可以在 ESP32 cluster 上面也做同樣的事情：


```python
gp = group([tasks.add.s(2, 2), tasks.add.s(4, 4)])
gp.get()
```

    
    Sending 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:30.147152 216'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:30.147152 216'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:30.147152 216'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:30.147152 216'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '17744'), ('function', 'dequeue_task'), ('message_id', '17744'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '17744'), ('message_id', '2018-04-05 17:48:30.547257 235'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:30.147086 215', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:30.147086 215', 'task_id': '2018-04-05 17:48:30.147086 215'}, 1)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '17744'), ('message_id', '2018-04-05 17:48:30.547257 235'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:30.147086 215', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:30.147086 215', 'task_id': '2018-04-05 17:48:30.147086 215'}, 1)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '17809'), ('function', 'dequeue_task'), ('message_id', '17809'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '17809'), ('message_id', '2018-04-05 17:48:30.656375 240'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:30.198165 218', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:30.198165 218', 'task_id': '2018-04-05 17:48:30.198165 218'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '17809'), ('message_id', '2018-04-05 17:48:30.656375 240'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:30.198165 218', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:30.198165 218', 'task_id': '2018-04-05 17:48:30.198165 218'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '18199'), ('function', 'dequeue_task'), ('message_id', '18199'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '18199'), ('message_id', '2018-04-05 17:48:30.912940 251'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '18199'), ('message_id', '2018-04-05 17:48:30.912940 251'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '18290'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 4, 'function': 'tasks.add', 'args': [2, 2], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:30.147086 215', 'message_type': 'result', 'task_id': '2018-04-05 17:48:30.147086 215', 'correlation_id': '2018-04-05 17:48:30.147086 215', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '18290'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '18394'), ('function', 'dequeue_task'), ('message_id', '18394'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '18394'), ('message_id', '2018-04-05 17:48:31.217117 265'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '18394'), ('message_id', '2018-04-05 17:48:31.217117 265'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '18400'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 8, 'function': 'tasks.add', 'args': [4, 4], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:30.198165 218', 'message_type': 'result', 'task_id': '2018-04-05 17:48:30.198165 218', 'correlation_id': '2018-04-05 17:48:30.198165 218', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '18400'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    




    [4, 8]



我們可以用 iterators:
```
>>> group(add.s(i, i) for i in xrange(10))()
```


```python
gp = group([tasks.add.s(i, i) for i in range(10)])
gp.get()
```

    
    Sending 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:31.363503 273'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:31.363503 273'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:31.363503 273'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:31.363503 273'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '19033'), ('function', 'dequeue_task'), ('message_id', '19033'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '19033'), ('message_id', '2018-04-05 17:48:31.799223 300'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.363450 272', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.363450 272', 'task_id': '2018-04-05 17:48:31.363450 272'}, 9)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '19033'), ('message_id', '2018-04-05 17:48:31.799223 300'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.363450 272', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.363450 272', 'task_id': '2018-04-05 17:48:31.363450 272'}, 9)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '19041'), ('function', 'dequeue_task'), ('message_id', '19041'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '19041'), ('message_id', '2018-04-05 17:48:31.891971 304'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408765 275', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408765 275', 'task_id': '2018-04-05 17:48:31.408765 275'}, 8)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '19041'), ('message_id', '2018-04-05 17:48:31.891971 304'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408765 275', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408765 275', 'task_id': '2018-04-05 17:48:31.408765 275'}, 8)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '19194'), ('function', 'dequeue_task'), ('message_id', '19194'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '19194'), ('message_id', '2018-04-05 17:48:32.076580 311'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408817 276', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408817 276', 'task_id': '2018-04-05 17:48:31.408817 276'}, 7)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '19194'), ('message_id', '2018-04-05 17:48:32.076580 311'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408817 276', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408817 276', 'task_id': '2018-04-05 17:48:31.408817 276'}, 7)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '19356'), ('function', 'dequeue_task'), ('message_id', '19356'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '19356'), ('message_id', '2018-04-05 17:48:32.198332 316'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408847 277', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408847 277', 'task_id': '2018-04-05 17:48:31.408847 277'}, 6)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '19356'), ('message_id', '2018-04-05 17:48:32.198332 316'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408847 277', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408847 277', 'task_id': '2018-04-05 17:48:31.408847 277'}, 6)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '19510'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 0, 'function': 'tasks.add', 'args': [0, 0], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.363450 272', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.363450 272', 'correlation_id': '2018-04-05 17:48:31.363450 272', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '19510'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '19613'), ('function', 'dequeue_task'), ('message_id', '19613'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '19613'), ('message_id', '2018-04-05 17:48:32.444144 327'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408868 278', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408868 278', 'task_id': '2018-04-05 17:48:31.408868 278'}, 5)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '19613'), ('message_id', '2018-04-05 17:48:32.444144 327'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408868 278', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408868 278', 'task_id': '2018-04-05 17:48:31.408868 278'}, 5)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '19597'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 2, 'function': 'tasks.add', 'args': [1, 1], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.408765 275', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.408765 275', 'correlation_id': '2018-04-05 17:48:31.408765 275', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '19597'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '19701'), ('function', 'dequeue_task'), ('message_id', '19701'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '19701'), ('message_id', '2018-04-05 17:48:32.730661 339'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408888 279', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408888 279', 'task_id': '2018-04-05 17:48:31.408888 279'}, 4)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '19701'), ('message_id', '2018-04-05 17:48:32.730661 339'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408888 279', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408888 279', 'task_id': '2018-04-05 17:48:31.408888 279'}, 4)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '19497'), ('function', 'dequeue_task'), ('message_id', '19497'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '19497'), ('message_id', '2018-04-05 17:48:32.836405 344'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408907 280', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408907 280', 'task_id': '2018-04-05 17:48:31.408907 280'}, 3)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '19497'), ('message_id', '2018-04-05 17:48:32.836405 344'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408907 280', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408907 280', 'task_id': '2018-04-05 17:48:31.408907 280'}, 3)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '19977'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 6, 'function': 'tasks.add', 'args': [3, 3], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.408847 277', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.408847 277', 'correlation_id': '2018-04-05 17:48:31.408847 277', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '19977'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '20080'), ('function', 'dequeue_task'), ('message_id', '20080'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '20080'), ('message_id', '2018-04-05 17:48:33.053487 354'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408927 281', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408927 281', 'task_id': '2018-04-05 17:48:31.408927 281'}, 2)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '20080'), ('message_id', '2018-04-05 17:48:33.053487 354'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408927 281', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408927 281', 'task_id': '2018-04-05 17:48:31.408927 281'}, 2)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '20304'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 8, 'function': 'tasks.add', 'args': [4, 4], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.408868 278', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.408868 278', 'correlation_id': '2018-04-05 17:48:31.408868 278', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '20304'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '20408'), ('function', 'dequeue_task'), ('message_id', '20408'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '20408'), ('message_id', '2018-04-05 17:48:33.331572 366'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408945 282', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408945 282', 'task_id': '2018-04-05 17:48:31.408945 282'}, 1)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '20408'), ('message_id', '2018-04-05 17:48:33.331572 366'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408945 282', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408945 282', 'task_id': '2018-04-05 17:48:31.408945 282'}, 1)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '20517'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 10, 'function': 'tasks.add', 'args': [5, 5], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.408888 279', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.408888 279', 'correlation_id': '2018-04-05 17:48:31.408888 279', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '20517'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '20622'), ('function', 'dequeue_task'), ('message_id', '20622'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '20622'), ('message_id', '2018-04-05 17:48:33.614941 378'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408964 283', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408964 283', 'task_id': '2018-04-05 17:48:31.408964 283'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '20622'), ('message_id', '2018-04-05 17:48:33.614941 378'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:31.408964 283', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:31.408964 283', 'task_id': '2018-04-05 17:48:31.408964 283'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '20530'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 12, 'function': 'tasks.add', 'args': [6, 6], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.408907 280', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.408907 280', 'correlation_id': '2018-04-05 17:48:31.408907 280', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '20530'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '20634'), ('function', 'dequeue_task'), ('message_id', '20634'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '20634'), ('message_id', '2018-04-05 17:48:33.895365 390'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '20634'), ('message_id', '2018-04-05 17:48:33.895365 390'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '20826'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 14, 'function': 'tasks.add', 'args': [7, 7], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.408927 281', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.408927 281', 'correlation_id': '2018-04-05 17:48:31.408927 281', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '20826'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '20930'), ('function', 'dequeue_task'), ('message_id', '20930'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '20930'), ('message_id', '2018-04-05 17:48:34.183259 403'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '20930'), ('message_id', '2018-04-05 17:48:34.183259 403'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '21150'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 16, 'function': 'tasks.add', 'args': [8, 8], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.408945 282', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.408945 282', 'correlation_id': '2018-04-05 17:48:31.408945 282', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '21150'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '21254'), ('function', 'dequeue_task'), ('message_id', '21254'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '21254'), ('message_id', '2018-04-05 17:48:34.421734 413'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '21254'), ('message_id', '2018-04-05 17:48:34.421734 413'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '21441'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 18, 'function': 'tasks.add', 'args': [9, 9], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.408964 283', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.408964 283', 'correlation_id': '2018-04-05 17:48:31.408964 283', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '21441'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '22003'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 4, 'function': 'tasks.add', 'args': [2, 2], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:31.408817 276', 'message_type': 'result', 'task_id': '2018-04-05 17:48:31.408817 276', 'correlation_id': '2018-04-05 17:48:31.408817 276', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '22003'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    




    [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]



### Chords
Celery [Chords](http://docs.celeryproject.org/en/latest/userguide/canvas.html#chords) 的主要作用是由兩段運算所組成的，第一段是一個`Groups`運算，其運算的結果會傳給第二段中的運算，作為其運算所需的參數。  

其作用可以用以下的例子來說明，`header`運算的結果會傳給`callback`做進一步的處理：
```
>>> callback = tsum.s()
>>> header = [add.s(i, i) for i in range(10)]
>>> result = chord(header)(callback)
>>> result.get()
9900
```
我們可以在 ESP32 cluster 上面也做同樣的事情：


```python
callback = tasks.xsum.s()
header = [tasks.add.s(i, i) for i in range(10)]
async_result = chord(header)(callback)
async_result.get()
```

    
    Sending 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:34.720288 429'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:34.720288 429'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '22106'), ('function', 'dequeue_task'), ('message_id', '22106'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '22106'), ('message_id', '2018-04-05 17:48:34.867152 443'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.720237 428', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.720237 428', 'task_id': '2018-04-05 17:48:34.720237 428'}, 9)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '22106'), ('message_id', '2018-04-05 17:48:34.867152 443'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.720237 428', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.720237 428', 'task_id': '2018-04-05 17:48:34.720237 428'}, 9)), ('sender', 'Client_366')])
    
    
    Data received: 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:34.720288 429'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:34.720288 429'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '22523'), ('function', 'dequeue_task'), ('message_id', '22523'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '22523'), ('message_id', '2018-04-05 17:48:35.469197 471'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.775980 432', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.775980 432', 'task_id': '2018-04-05 17:48:34.775980 432'}, 8)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '22523'), ('message_id', '2018-04-05 17:48:35.469197 471'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.775980 432', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.775980 432', 'task_id': '2018-04-05 17:48:34.775980 432'}, 8)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '22428'), ('function', 'dequeue_task'), ('message_id', '22428'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '22428'), ('message_id', '2018-04-05 17:48:35.584851 476'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776030 433', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776030 433', 'task_id': '2018-04-05 17:48:34.776030 433'}, 7)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '22428'), ('message_id', '2018-04-05 17:48:35.584851 476'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776030 433', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776030 433', 'task_id': '2018-04-05 17:48:34.776030 433'}, 7)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '22486'), ('function', 'dequeue_task'), ('message_id', '22486'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '22486'), ('message_id', '2018-04-05 17:48:35.666772 479'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776084 434', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776084 434', 'task_id': '2018-04-05 17:48:34.776084 434'}, 6)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '22486'), ('message_id', '2018-04-05 17:48:35.666772 479'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776084 434', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776084 434', 'task_id': '2018-04-05 17:48:34.776084 434'}, 6)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '22492'), ('function', 'dequeue_task'), ('message_id', '22492'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '22492'), ('message_id', '2018-04-05 17:48:35.808710 485'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776118 435', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776118 435', 'task_id': '2018-04-05 17:48:34.776118 435'}, 5)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '22492'), ('message_id', '2018-04-05 17:48:35.808710 485'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776118 435', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776118 435', 'task_id': '2018-04-05 17:48:34.776118 435'}, 5)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '23313'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 2, 'function': 'tasks.add', 'args': [1, 1], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.775980 432', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.775980 432', 'correlation_id': '2018-04-05 17:48:34.775980 432', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '23313'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '23220'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 4, 'function': 'tasks.add', 'args': [2, 2], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.776030 433', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.776030 433', 'correlation_id': '2018-04-05 17:48:34.776030 433', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '23220'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '23417'), ('function', 'dequeue_task'), ('message_id', '23417'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '23417'), ('message_id', '2018-04-05 17:48:36.341813 505'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776143 436', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776143 436', 'task_id': '2018-04-05 17:48:34.776143 436'}, 4)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '23417'), ('message_id', '2018-04-05 17:48:36.341813 505'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776143 436', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776143 436', 'task_id': '2018-04-05 17:48:34.776143 436'}, 4)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '23324'), ('function', 'dequeue_task'), ('message_id', '23324'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '23324'), ('message_id', '2018-04-05 17:48:36.464812 510'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776165 437', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776165 437', 'task_id': '2018-04-05 17:48:34.776165 437'}, 3)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '23324'), ('message_id', '2018-04-05 17:48:36.464812 510'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776165 437', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776165 437', 'task_id': '2018-04-05 17:48:34.776165 437'}, 3)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '23707'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 6, 'function': 'tasks.add', 'args': [3, 3], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.776084 434', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.776084 434', 'correlation_id': '2018-04-05 17:48:34.776084 434', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '23707'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '23810'), ('function', 'dequeue_task'), ('message_id', '23810'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '23810'), ('message_id', '2018-04-05 17:48:36.708569 520'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776184 438', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776184 438', 'task_id': '2018-04-05 17:48:34.776184 438'}, 2)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '23810'), ('message_id', '2018-04-05 17:48:36.708569 520'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776184 438', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776184 438', 'task_id': '2018-04-05 17:48:34.776184 438'}, 2)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '23876'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 8, 'function': 'tasks.add', 'args': [4, 4], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.776118 435', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.776118 435', 'correlation_id': '2018-04-05 17:48:34.776118 435', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '23876'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '23981'), ('function', 'dequeue_task'), ('message_id', '23981'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '23981'), ('message_id', '2018-04-05 17:48:36.984051 532'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776203 439', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776203 439', 'task_id': '2018-04-05 17:48:34.776203 439'}, 1)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '23981'), ('message_id', '2018-04-05 17:48:36.984051 532'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776203 439', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776203 439', 'task_id': '2018-04-05 17:48:34.776203 439'}, 1)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '24129'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 12, 'function': 'tasks.add', 'args': [6, 6], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.776165 437', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.776165 437', 'correlation_id': '2018-04-05 17:48:34.776165 437', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '24129'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '24233'), ('function', 'dequeue_task'), ('message_id', '24233'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '24233'), ('message_id', '2018-04-05 17:48:37.229212 542'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776226 440', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776226 440', 'task_id': '2018-04-05 17:48:34.776226 440'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '24233'), ('message_id', '2018-04-05 17:48:37.229212 542'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:34.776226 440', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:34.776226 440', 'task_id': '2018-04-05 17:48:34.776226 440'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '24223'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 10, 'function': 'tasks.add', 'args': [5, 5], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.776143 436', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.776143 436', 'correlation_id': '2018-04-05 17:48:34.776143 436', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '24223'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '24326'), ('function', 'dequeue_task'), ('message_id', '24326'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '24326'), ('message_id', '2018-04-05 17:48:37.482078 553'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '24326'), ('message_id', '2018-04-05 17:48:37.482078 553'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '24561'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 14, 'function': 'tasks.add', 'args': [7, 7], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.776184 438', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.776184 438', 'correlation_id': '2018-04-05 17:48:34.776184 438', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '24561'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '24667'), ('function', 'dequeue_task'), ('message_id', '24667'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '24667'), ('message_id', '2018-04-05 17:48:37.731619 564'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '24667'), ('message_id', '2018-04-05 17:48:37.731619 564'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    Timeout: no result returned for request with id 2018-04-05 17:48:34.720237 428
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '24796'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 16, 'function': 'tasks.add', 'args': [8, 8], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.776203 439', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.776203 439', 'correlation_id': '2018-04-05 17:48:34.776203 439', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '24796'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '24900'), ('function', 'dequeue_task'), ('message_id', '24900'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '24900'), ('message_id', '2018-04-05 17:48:37.990641 576'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '24900'), ('message_id', '2018-04-05 17:48:37.990641 576'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '25325'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 0, 'function': 'tasks.add', 'args': [0, 0], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.720237 428', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.720237 428', 'correlation_id': '2018-04-05 17:48:34.720237 428', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '25325'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '25431'), ('function', 'dequeue_task'), ('message_id', '25431'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '25431'), ('message_id', '2018-04-05 17:48:38.240552 587'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '25431'), ('message_id', '2018-04-05 17:48:38.240552 587'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '24973'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 18, 'function': 'tasks.add', 'args': [9, 9], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:34.776226 440', 'message_type': 'result', 'task_id': '2018-04-05 17:48:34.776226 440', 'correlation_id': '2018-04-05 17:48:34.776226 440', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '24973'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Sending 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:40.661880 719'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:40.661880 719'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:40.661880 719'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:40.661880 719'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '28435'), ('function', 'dequeue_task'), ('message_id', '28435'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '28435'), ('message_id', '2018-04-05 17:48:41.297656 750'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:40.661840 718', 'function': 'tasks.xsum', 'args': ([None, 2, 4, 6, 8, 10, 12, 14, 16, 18],), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:40.661840 718', 'task_id': '2018-04-05 17:48:40.661840 718'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 524 bytes
    Message:
    OrderedDict([('correlation_id', '28435'), ('message_id', '2018-04-05 17:48:41.297656 750'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:40.661840 718', 'function': 'tasks.xsum', 'args': ([None, 2, 4, 6, 8, 10, 12, 14, 16, 18],), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:40.661840 718', 'task_id': '2018-04-05 17:48:40.661840 718'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '28341'), ('function', 'dequeue_task'), ('message_id', '28341'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '28341'), ('message_id', '2018-04-05 17:48:41.406124 755'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '28341'), ('message_id', '2018-04-05 17:48:41.406124 755'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '28400'), ('function', 'dequeue_task'), ('message_id', '28400'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '28400'), ('message_id', '2018-04-05 17:48:41.523419 760'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '28400'), ('message_id', '2018-04-05 17:48:41.523419 760'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '28406'), ('function', 'dequeue_task'), ('message_id', '28406'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '28406'), ('message_id', '2018-04-05 17:48:41.660699 766'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 208 bytes
    Message:
    OrderedDict([('correlation_id', '28406'), ('message_id', '2018-04-05 17:48:41.660699 766'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    Timeout: no result returned for request with id 2018-04-05 17:48:40.661840 718
    

上述的運算可以直接寫成：
```
chord(add.s(i, i) for i in xrange(10))(tsum.s()).get()
```
我們可以在 ESP32 cluster 上面也做同樣的事情：


```python
async_result = chord([tasks.add.s(i, i) for i in range(10)])(tasks.xsum.s())
async_result.get()
```

    
    Sending 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:43.720981 882'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:43.720981 882'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 259 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:43.720981 882'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:43.720981 882'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '31484'), ('function', 'dequeue_task'), ('message_id', '31484'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '31484'), ('message_id', '2018-04-05 17:48:44.417908 924'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.720915 881', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.720915 881', 'task_id': '2018-04-05 17:48:43.720915 881'}, 9)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '31484'), ('message_id', '2018-04-05 17:48:44.417908 924'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.720915 881', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.720915 881', 'task_id': '2018-04-05 17:48:43.720915 881'}, 9)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '31388'), ('function', 'dequeue_task'), ('message_id', '31388'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '31388'), ('message_id', '2018-04-05 17:48:44.542577 930'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786401 884', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786401 884', 'task_id': '2018-04-05 17:48:43.786401 884'}, 8)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '31388'), ('message_id', '2018-04-05 17:48:44.542577 930'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786401 884', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786401 884', 'task_id': '2018-04-05 17:48:43.786401 884'}, 8)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '31447'), ('function', 'dequeue_task'), ('message_id', '31447'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '31447'), ('message_id', '2018-04-05 17:48:44.667264 935'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786497 885', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786497 885', 'task_id': '2018-04-05 17:48:43.786497 885'}, 7)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '31447'), ('message_id', '2018-04-05 17:48:44.667264 935'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786497 885', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786497 885', 'task_id': '2018-04-05 17:48:43.786497 885'}, 7)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '31456'), ('function', 'dequeue_task'), ('message_id', '31456'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '31456'), ('message_id', '2018-04-05 17:48:44.778402 940'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786534 886', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786534 886', 'task_id': '2018-04-05 17:48:43.786534 886'}, 6)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '31456'), ('message_id', '2018-04-05 17:48:44.778402 940'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786534 886', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786534 886', 'task_id': '2018-04-05 17:48:43.786534 886'}, 6)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '32352'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 0, 'function': 'tasks.add', 'args': [0, 0], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.720915 881', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.720915 881', 'correlation_id': '2018-04-05 17:48:43.720915 881', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '32352'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '32259'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 2, 'function': 'tasks.add', 'args': [1, 1], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.786401 884', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.786401 884', 'correlation_id': '2018-04-05 17:48:43.786401 884', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '32259'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '32455'), ('function', 'dequeue_task'), ('message_id', '32455'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '32455'), ('message_id', '2018-04-05 17:48:45.504842 976'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786563 887', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786563 887', 'task_id': '2018-04-05 17:48:43.786563 887'}, 5)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '32455'), ('message_id', '2018-04-05 17:48:45.504842 976'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786563 887', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786563 887', 'task_id': '2018-04-05 17:48:43.786563 887'}, 5)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '32363'), ('function', 'dequeue_task'), ('message_id', '32363'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '32363'), ('message_id', '2018-04-05 17:48:45.626279 981'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786594 888', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786594 888', 'task_id': '2018-04-05 17:48:43.786594 888'}, 4)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '32363'), ('message_id', '2018-04-05 17:48:45.626279 981'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786594 888', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786594 888', 'task_id': '2018-04-05 17:48:43.786594 888'}, 4)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '32388'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 4, 'function': 'tasks.add', 'args': [2, 2], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.786497 885', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.786497 885', 'correlation_id': '2018-04-05 17:48:43.786497 885', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '32388'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '32492'), ('function', 'dequeue_task'), ('message_id', '32492'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '32492'), ('message_id', '2018-04-05 17:48:45.913267 992'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786616 889', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786616 889', 'task_id': '2018-04-05 17:48:43.786616 889'}, 3)), ('sender', 'Client_366')])
    
    
    Sending 489 bytes
    Message:
    OrderedDict([('correlation_id', '32492'), ('message_id', '2018-04-05 17:48:45.913267 992'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786616 889', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786616 889', 'task_id': '2018-04-05 17:48:43.786616 889'}, 3)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '32517'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 6, 'function': 'tasks.add', 'args': [3, 3], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.786534 886', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.786534 886', 'correlation_id': '2018-04-05 17:48:43.786534 886', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '32517'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '32620'), ('function', 'dequeue_task'), ('message_id', '32620'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '32620'), ('message_id', '2018-04-05 17:48:46.182368 1004'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786634 890', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786634 890', 'task_id': '2018-04-05 17:48:43.786634 890'}, 2)), ('sender', 'Client_366')])
    
    
    Sending 490 bytes
    Message:
    OrderedDict([('correlation_id', '32620'), ('message_id', '2018-04-05 17:48:46.182368 1004'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786634 890', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786634 890', 'task_id': '2018-04-05 17:48:43.786634 890'}, 2)), ('sender', 'Client_366')])
    
    
    Data received: 525 bytes
    Message:
    OrderedDict([('correlation_id', '33393'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 8, 'function': 'tasks.add', 'args': [4, 4], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.786563 887', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.786563 887', 'correlation_id': '2018-04-05 17:48:43.786563 887', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '33393'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '33498'), ('function', 'dequeue_task'), ('message_id', '33498'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '33498'), ('message_id', '2018-04-05 17:48:46.402346 1014'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786651 891', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786651 891', 'task_id': '2018-04-05 17:48:43.786651 891'}, 1)), ('sender', 'Client_366')])
    
    
    Sending 490 bytes
    Message:
    OrderedDict([('correlation_id', '33498'), ('message_id', '2018-04-05 17:48:46.402346 1014'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786651 891', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786651 891', 'task_id': '2018-04-05 17:48:43.786651 891'}, 1)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '33655'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 10, 'function': 'tasks.add', 'args': [5, 5], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.786594 888', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.786594 888', 'correlation_id': '2018-04-05 17:48:43.786594 888', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '33655'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '33759'), ('function', 'dequeue_task'), ('message_id', '33759'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '33759'), ('message_id', '2018-04-05 17:48:46.605582 1023'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786670 892', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786670 892', 'task_id': '2018-04-05 17:48:43.786670 892'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 490 bytes
    Message:
    OrderedDict([('correlation_id', '33759'), ('message_id', '2018-04-05 17:48:46.605582 1023'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:43.786670 892', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:43.786670 892', 'task_id': '2018-04-05 17:48:43.786670 892'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '34208'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 12, 'function': 'tasks.add', 'args': [6, 6], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.786616 889', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.786616 889', 'correlation_id': '2018-04-05 17:48:43.786616 889', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '34208'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '34313'), ('function', 'dequeue_task'), ('message_id', '34313'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '34313'), ('message_id', '2018-04-05 17:48:47.243802 1055'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '34313'), ('message_id', '2018-04-05 17:48:47.243802 1055'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '34244'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 16, 'function': 'tasks.add', 'args': [8, 8], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.786651 891', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.786651 891', 'correlation_id': '2018-04-05 17:48:43.786651 891', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '34244'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '34349'), ('function', 'dequeue_task'), ('message_id', '34349'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '34349'), ('message_id', '2018-04-05 17:48:47.496301 1065'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '34349'), ('message_id', '2018-04-05 17:48:47.496301 1065'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '33995'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 14, 'function': 'tasks.add', 'args': [7, 7], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.786634 890', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.786634 890', 'correlation_id': '2018-04-05 17:48:43.786634 890', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '33995'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '34101'), ('function', 'dequeue_task'), ('message_id', '34101'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '34101'), ('message_id', '2018-04-05 17:48:47.773575 1078'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '34101'), ('message_id', '2018-04-05 17:48:47.773575 1078'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 526 bytes
    Message:
    OrderedDict([('correlation_id', '35054'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 18, 'function': 'tasks.add', 'args': [9, 9], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:43.786670 892', 'message_type': 'result', 'task_id': '2018-04-05 17:48:43.786670 892', 'correlation_id': '2018-04-05 17:48:43.786670 892', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '35054'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Sending 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:47.962733 1089'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:47.962733 1089'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:47.962733 1089'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:47.962733 1089'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '35682'), ('function', 'dequeue_task'), ('message_id', '35682'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '35682'), ('message_id', '2018-04-05 17:48:48.381118 1108'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:47.962694 1088', 'function': 'tasks.xsum', 'args': ([0, 2, 4, 6, 8, 10, 12, 14, 16, 18],), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:47.962694 1088', 'task_id': '2018-04-05 17:48:47.962694 1088'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 525 bytes
    Message:
    OrderedDict([('correlation_id', '35682'), ('message_id', '2018-04-05 17:48:48.381118 1108'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:47.962694 1088', 'function': 'tasks.xsum', 'args': ([0, 2, 4, 6, 8, 10, 12, 14, 16, 18],), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:47.962694 1088', 'task_id': '2018-04-05 17:48:47.962694 1088'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '35643'), ('function', 'dequeue_task'), ('message_id', '35643'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '35643'), ('message_id', '2018-04-05 17:48:48.524834 1115'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '35643'), ('message_id', '2018-04-05 17:48:48.524834 1115'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 561 bytes
    Message:
    OrderedDict([('correlation_id', '36276'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 90, 'function': 'tasks.xsum', 'args': [[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:47.962694 1088', 'message_type': 'result', 'task_id': '2018-04-05 17:48:47.962694 1088', 'correlation_id': '2018-04-05 17:48:47.962694 1088', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '36276'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    




    90



### Map & Starmap
Celery [Map & Starmap](http://docs.celeryproject.org/en/latest/userguide/canvas.html#map-starmap) 的主要作用和 Python 中的`map`指令一樣，會對一個 list 中的每個 element 做指定的運算，例如下例會分別對`range(10)`,`range(100)`做`sum`運算：
```
>>> ~xsum.map([range(10), range(100)])
[45, 4950]
```
我們可以在 ESP32 cluster 上面也做同樣的事情，但是須先使用`list()`對`range`物件做展開：


```python
gp = tasks.xsum.map([list(range(10)), list(range(100))])
gp.get()
```

    
    Sending 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:49.449756 1163'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:49.449756 1163'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '35585'), ('function', 'dequeue_task'), ('message_id', '35585'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '35585'), ('message_id', '2018-04-05 17:48:49.759007 1173'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:49.449700 1162', 'function': 'tasks.xsum', 'args': ([0, 1, 2, 3, 4, 5, 6, 7, 8, 9],), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:49.449700 1162', 'task_id': '2018-04-05 17:48:49.449700 1162'}, 1)), ('sender', 'Client_366')])
    
    
    Sending 520 bytes
    Message:
    OrderedDict([('correlation_id', '35585'), ('message_id', '2018-04-05 17:48:49.759007 1173'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:49.449700 1162', 'function': 'tasks.xsum', 'args': ([0, 1, 2, 3, 4, 5, 6, 7, 8, 9],), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:49.449700 1162', 'task_id': '2018-04-05 17:48:49.449700 1162'}, 1)), ('sender', 'Client_366')])
    
    
    Data received: 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:49.449756 1163'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:49.449756 1163'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '37313'), ('function', 'dequeue_task'), ('message_id', '37313'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '37313'), ('message_id', '2018-04-05 17:48:50.072712 1183'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:49.548137 1165', 'function': 'tasks.xsum', 'args': ([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99],), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:49.548137 1165', 'task_id': '2018-04-05 17:48:49.548137 1165'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 880 bytes
    Message:
    OrderedDict([('correlation_id', '37313'), ('message_id', '2018-04-05 17:48:50.072712 1183'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:49.548137 1165', 'function': 'tasks.xsum', 'args': ([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99],), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:49.548137 1165', 'task_id': '2018-04-05 17:48:49.548137 1165'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '37277'), ('function', 'dequeue_task'), ('message_id', '37277'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '37277'), ('message_id', '2018-04-05 17:48:50.198705 1188'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '37277'), ('message_id', '2018-04-05 17:48:50.198705 1188'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '37543'), ('function', 'dequeue_task'), ('message_id', '37543'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '37543'), ('message_id', '2018-04-05 17:48:50.403895 1198'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '37543'), ('message_id', '2018-04-05 17:48:50.403895 1198'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 918 bytes
    Message:
    OrderedDict([('correlation_id', '38082'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 4950, 'function': 'tasks.xsum', 'args': [[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99]], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:49.548137 1165', 'message_type': 'result', 'task_id': '2018-04-05 17:48:49.548137 1165', 'correlation_id': '2018-04-05 17:48:49.548137 1165', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '38082'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 556 bytes
    Message:
    OrderedDict([('correlation_id', '38088'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 45, 'function': 'tasks.xsum', 'args': [[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:49.449700 1162', 'message_type': 'result', 'task_id': '2018-04-05 17:48:49.449700 1162', 'correlation_id': '2018-04-05 17:48:49.449700 1162', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '38088'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    




    [45, 4950]



    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '38197'), ('function', 'dequeue_task'), ('message_id', '38197'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    

`starmap`的作用和`map`指令一樣，會對一個 list 中的每個 element 做指定的運算，只是會先做 star展開，將一個`list`展開成為 positional arguments：
```
>>> ~add.starmap(zip(range(10), range(10)))
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```
我們可以在 ESP32 cluster 上面也做同樣的事情，但是須先使用`list()`對`zip`物件做展開：


```python
gp = tasks.add.starmap(list(zip(range(10), range(10))))
gp.get()
```

    
    Processed result:
    OrderedDict([('correlation_id', '38197'), ('message_id', '2018-04-05 17:48:51.273174 1234'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '38197'), ('message_id', '2018-04-05 17:48:51.273174 1234'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:51.314332 1237'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:51.314332 1237'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:51.314332 1237'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:51.314332 1237'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '39176'), ('function', 'dequeue_task'), ('message_id', '39176'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '39176'), ('message_id', '2018-04-05 17:48:52.122649 1268'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.314278 1236', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.314278 1236', 'task_id': '2018-04-05 17:48:51.314278 1236'}, 9)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '39176'), ('message_id', '2018-04-05 17:48:52.122649 1268'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.314278 1236', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.314278 1236', 'task_id': '2018-04-05 17:48:51.314278 1236'}, 9)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '39037'), ('function', 'dequeue_task'), ('message_id', '39037'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '39037'), ('message_id', '2018-04-05 17:48:52.232293 1273'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.415903 1239', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.415903 1239', 'task_id': '2018-04-05 17:48:51.415903 1239'}, 8)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '39037'), ('message_id', '2018-04-05 17:48:52.232293 1273'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.415903 1239', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.415903 1239', 'task_id': '2018-04-05 17:48:51.415903 1239'}, 8)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '39991'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 2, 'function': 'tasks.add', 'args': [1, 1], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.415903 1239', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.415903 1239', 'correlation_id': '2018-04-05 17:48:51.415903 1239', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '39991'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '40095'), ('function', 'dequeue_task'), ('message_id', '40095'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '40095'), ('message_id', '2018-04-05 17:48:53.199953 1311'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.415961 1240', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.415961 1240', 'task_id': '2018-04-05 17:48:51.415961 1240'}, 7)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '40095'), ('message_id', '2018-04-05 17:48:53.199953 1311'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.415961 1240', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.415961 1240', 'task_id': '2018-04-05 17:48:51.415961 1240'}, 7)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '39934'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 0, 'function': 'tasks.add', 'args': [0, 0], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.314278 1236', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.314278 1236', 'correlation_id': '2018-04-05 17:48:51.314278 1236', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '39934'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '40038'), ('function', 'dequeue_task'), ('message_id', '40038'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '40038'), ('message_id', '2018-04-05 17:48:53.454767 1320'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.415987 1241', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.415987 1241', 'task_id': '2018-04-05 17:48:51.415987 1241'}, 6)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '40038'), ('message_id', '2018-04-05 17:48:53.454767 1320'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.415987 1241', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.415987 1241', 'task_id': '2018-04-05 17:48:51.415987 1241'}, 6)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '39072'), ('function', 'dequeue_task'), ('message_id', '39072'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '39072'), ('message_id', '2018-04-05 17:48:53.650921 1324'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416008 1242', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416008 1242', 'task_id': '2018-04-05 17:48:51.416008 1242'}, 5)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '39072'), ('message_id', '2018-04-05 17:48:53.650921 1324'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416008 1242', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416008 1242', 'task_id': '2018-04-05 17:48:51.416008 1242'}, 5)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '41171'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 6, 'function': 'tasks.add', 'args': [3, 3], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.415987 1241', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.415987 1241', 'correlation_id': '2018-04-05 17:48:51.415987 1241', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '41171'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '41274'), ('function', 'dequeue_task'), ('message_id', '41274'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '41274'), ('message_id', '2018-04-05 17:48:54.224012 1339'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416028 1243', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416028 1243', 'task_id': '2018-04-05 17:48:51.416028 1243'}, 4)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '41274'), ('message_id', '2018-04-05 17:48:54.224012 1339'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416028 1243', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416028 1243', 'task_id': '2018-04-05 17:48:51.416028 1243'}, 4)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '41054'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 4, 'function': 'tasks.add', 'args': [2, 2], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.415961 1240', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.415961 1240', 'correlation_id': '2018-04-05 17:48:51.415961 1240', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '41054'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '41159'), ('function', 'dequeue_task'), ('message_id', '41159'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '41159'), ('message_id', '2018-04-05 17:48:54.595576 1349'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416048 1244', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416048 1244', 'task_id': '2018-04-05 17:48:51.416048 1244'}, 3)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '41159'), ('message_id', '2018-04-05 17:48:54.595576 1349'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416048 1244', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416048 1244', 'task_id': '2018-04-05 17:48:51.416048 1244'}, 3)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '41378'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 8, 'function': 'tasks.add', 'args': [4, 4], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.416008 1242', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.416008 1242', 'correlation_id': '2018-04-05 17:48:51.416008 1242', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '41378'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '41482'), ('function', 'dequeue_task'), ('message_id', '41482'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '41482'), ('message_id', '2018-04-05 17:48:55.097132 1359'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416066 1245', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416066 1245', 'task_id': '2018-04-05 17:48:51.416066 1245'}, 2)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '41482'), ('message_id', '2018-04-05 17:48:55.097132 1359'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416066 1245', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416066 1245', 'task_id': '2018-04-05 17:48:51.416066 1245'}, 2)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '42988'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 14, 'function': 'tasks.add', 'args': [7, 7], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.416066 1245', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.416066 1245', 'correlation_id': '2018-04-05 17:48:51.416066 1245', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '42988'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '42572'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 10, 'function': 'tasks.add', 'args': [5, 5], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.416028 1243', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.416028 1243', 'correlation_id': '2018-04-05 17:48:51.416028 1243', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '42572'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '42676'), ('function', 'dequeue_task'), ('message_id', '42676'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '42676'), ('message_id', '2018-04-05 17:48:56.256766 1400'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416083 1246', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416083 1246', 'task_id': '2018-04-05 17:48:51.416083 1246'}, 1)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '42676'), ('message_id', '2018-04-05 17:48:56.256766 1400'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416083 1246', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416083 1246', 'task_id': '2018-04-05 17:48:51.416083 1246'}, 1)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '42631'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 12, 'function': 'tasks.add', 'args': [6, 6], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.416048 1244', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.416048 1244', 'correlation_id': '2018-04-05 17:48:51.416048 1244', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '42631'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '42736'), ('function', 'dequeue_task'), ('message_id', '42736'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '42736'), ('message_id', '2018-04-05 17:48:56.585402 1410'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416100 1247', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416100 1247', 'task_id': '2018-04-05 17:48:51.416100 1247'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 493 bytes
    Message:
    OrderedDict([('correlation_id', '42736'), ('message_id', '2018-04-05 17:48:56.585402 1410'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:51.416100 1247', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:51.416100 1247', 'task_id': '2018-04-05 17:48:51.416100 1247'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '43094'), ('function', 'dequeue_task'), ('message_id', '43094'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '43094'), ('message_id', '2018-04-05 17:48:56.743045 1416'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '43094'), ('message_id', '2018-04-05 17:48:56.743045 1416'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '44174'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 16, 'function': 'tasks.add', 'args': [8, 8], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.416083 1246', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.416083 1246', 'correlation_id': '2018-04-05 17:48:51.416083 1246', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '44174'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '44280'), ('function', 'dequeue_task'), ('message_id', '44280'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '44280'), ('message_id', '2018-04-05 17:48:57.384804 1441'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '44280'), ('message_id', '2018-04-05 17:48:57.384804 1441'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '44427'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 18, 'function': 'tasks.add', 'args': [9, 9], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:51.416100 1247', 'message_type': 'result', 'task_id': '2018-04-05 17:48:51.416100 1247', 'correlation_id': '2018-04-05 17:48:51.416100 1247', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '44427'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    




    [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]



### Chunks
Celery [Chunks](http://docs.celeryproject.org/en/latest/userguide/canvas.html#chunks) 的主要作用是把一大串的資料切成指定的份數，分發給遠端的 workers 協處處理，例如：
```
>>> res = add.chunks(zip(range(100), range(100)), 10)()
>>> res.get()
[[0, 2, 4, 6, 8, 10, 12, 14, 16, 18],
 [20, 22, 24, 26, 28, 30, 32, 34, 36, 38],
 [40, 42, 44, 46, 48, 50, 52, 54, 56, 58],
 [60, 62, 64, 66, 68, 70, 72, 74, 76, 78],
 [80, 82, 84, 86, 88, 90, 92, 94, 96, 98],
 [100, 102, 104, 106, 108, 110, 112, 114, 116, 118],
 [120, 122, 124, 126, 128, 130, 132, 134, 136, 138],
 [140, 142, 144, 146, 148, 150, 152, 154, 156, 158],
 [160, 162, 164, 166, 168, 170, 172, 174, 176, 178],
 [180, 182, 184, 186, 188, 190, 192, 194, 196, 198]]
```
我們可以在 ESP32 cluster 上面也做同樣的事情，但是須先使用`list()`對`zip`物件做展開：


```python
ck = tasks.add.chunks(list(zip(range(100), range(100))), 10)
async_result = ck()
async_result.get()
```

    
    Sending 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:57.559084 1450'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:57.559084 1450'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:48:57.559084 1450'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:48:57.559084 1450'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '45274'), ('function', 'dequeue_task'), ('message_id', '45274'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '45274'), ('message_id', '2018-04-05 17:48:58.023949 1562'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.559018 1449', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.559018 1449', 'task_id': '2018-04-05 17:48:57.559018 1449'}, 99)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '45274'), ('message_id', '2018-04-05 17:48:58.023949 1562'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.559018 1449', 'function': 'tasks.add', 'args': (0, 0), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.559018 1449', 'task_id': '2018-04-05 17:48:57.559018 1449'}, 99)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '45237'), ('function', 'dequeue_task'), ('message_id', '45237'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '45237'), ('message_id', '2018-04-05 17:48:58.154267 1567'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.628894 1452', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.628894 1452', 'task_id': '2018-04-05 17:48:57.628894 1452'}, 98)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '45237'), ('message_id', '2018-04-05 17:48:58.154267 1567'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.628894 1452', 'function': 'tasks.add', 'args': (1, 1), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.628894 1452', 'task_id': '2018-04-05 17:48:57.628894 1452'}, 98)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '45259'), ('function', 'dequeue_task'), ('message_id', '45259'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '45259'), ('message_id', '2018-04-05 17:48:58.315118 1573'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.628952 1453', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.628952 1453', 'task_id': '2018-04-05 17:48:57.628952 1453'}, 97)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '45259'), ('message_id', '2018-04-05 17:48:58.315118 1573'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.628952 1453', 'function': 'tasks.add', 'args': (2, 2), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.628952 1453', 'task_id': '2018-04-05 17:48:57.628952 1453'}, 97)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '45930'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 2, 'function': 'tasks.add', 'args': [1, 1], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.628894 1452', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.628894 1452', 'correlation_id': '2018-04-05 17:48:57.628894 1452', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '45930'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '46034'), ('function', 'dequeue_task'), ('message_id', '46034'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '46034'), ('message_id', '2018-04-05 17:48:58.931588 1604'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.628980 1454', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.628980 1454', 'task_id': '2018-04-05 17:48:57.628980 1454'}, 96)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '46034'), ('message_id', '2018-04-05 17:48:58.931588 1604'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.628980 1454', 'function': 'tasks.add', 'args': (3, 3), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.628980 1454', 'task_id': '2018-04-05 17:48:57.628980 1454'}, 96)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '45950'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 4, 'function': 'tasks.add', 'args': [2, 2], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.628952 1453', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.628952 1453', 'correlation_id': '2018-04-05 17:48:57.628952 1453', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '45950'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '46055'), ('function', 'dequeue_task'), ('message_id', '46055'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '46055'), ('message_id', '2018-04-05 17:48:59.174445 1615'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.628999 1455', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.628999 1455', 'task_id': '2018-04-05 17:48:57.628999 1455'}, 95)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '46055'), ('message_id', '2018-04-05 17:48:59.174445 1615'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.628999 1455', 'function': 'tasks.add', 'args': (4, 4), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.628999 1455', 'task_id': '2018-04-05 17:48:57.628999 1455'}, 95)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '45949'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 0, 'function': 'tasks.add', 'args': [0, 0], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.559018 1449', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.559018 1449', 'correlation_id': '2018-04-05 17:48:57.559018 1449', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '45949'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '46053'), ('function', 'dequeue_task'), ('message_id', '46053'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '46053'), ('message_id', '2018-04-05 17:48:59.559769 1625'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629020 1456', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629020 1456', 'task_id': '2018-04-05 17:48:57.629020 1456'}, 94)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '46053'), ('message_id', '2018-04-05 17:48:59.559769 1625'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629020 1456', 'function': 'tasks.add', 'args': (5, 5), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629020 1456', 'task_id': '2018-04-05 17:48:57.629020 1456'}, 94)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '46788'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 6, 'function': 'tasks.add', 'args': [3, 3], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.628980 1454', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.628980 1454', 'correlation_id': '2018-04-05 17:48:57.628980 1454', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '46788'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '46894'), ('function', 'dequeue_task'), ('message_id', '46894'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '46894'), ('message_id', '2018-04-05 17:48:59.902092 1635'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629039 1457', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629039 1457', 'task_id': '2018-04-05 17:48:57.629039 1457'}, 93)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '46894'), ('message_id', '2018-04-05 17:48:59.902092 1635'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629039 1457', 'function': 'tasks.add', 'args': (6, 6), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629039 1457', 'task_id': '2018-04-05 17:48:57.629039 1457'}, 93)), ('sender', 'Client_366')])
    
    
    Data received: 528 bytes
    Message:
    OrderedDict([('correlation_id', '46994'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 8, 'function': 'tasks.add', 'args': [4, 4], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.628999 1455', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.628999 1455', 'correlation_id': '2018-04-05 17:48:57.628999 1455', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '46994'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '47098'), ('function', 'dequeue_task'), ('message_id', '47098'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '47098'), ('message_id', '2018-04-05 17:49:00.229911 1646'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629056 1458', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629056 1458', 'task_id': '2018-04-05 17:48:57.629056 1458'}, 92)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '47098'), ('message_id', '2018-04-05 17:49:00.229911 1646'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629056 1458', 'function': 'tasks.add', 'args': (7, 7), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629056 1458', 'task_id': '2018-04-05 17:48:57.629056 1458'}, 92)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '47465'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 10, 'function': 'tasks.add', 'args': [5, 5], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629020 1456', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629020 1456', 'correlation_id': '2018-04-05 17:48:57.629020 1456', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '47465'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '47570'), ('function', 'dequeue_task'), ('message_id', '47570'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '47570'), ('message_id', '2018-04-05 17:49:00.520386 1658'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629076 1459', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629076 1459', 'task_id': '2018-04-05 17:48:57.629076 1459'}, 91)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '47570'), ('message_id', '2018-04-05 17:49:00.520386 1658'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629076 1459', 'function': 'tasks.add', 'args': (8, 8), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629076 1459', 'task_id': '2018-04-05 17:48:57.629076 1459'}, 91)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '47744'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 12, 'function': 'tasks.add', 'args': [6, 6], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629039 1457', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629039 1457', 'correlation_id': '2018-04-05 17:48:57.629039 1457', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '47744'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '47848'), ('function', 'dequeue_task'), ('message_id', '47848'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '47848'), ('message_id', '2018-04-05 17:49:00.723347 1666'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629095 1460', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629095 1460', 'task_id': '2018-04-05 17:48:57.629095 1460'}, 90)), ('sender', 'Client_366')])
    
    
    Sending 494 bytes
    Message:
    OrderedDict([('correlation_id', '47848'), ('message_id', '2018-04-05 17:49:00.723347 1666'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629095 1460', 'function': 'tasks.add', 'args': (9, 9), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629095 1460', 'task_id': '2018-04-05 17:48:57.629095 1460'}, 90)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '48265'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 16, 'function': 'tasks.add', 'args': [8, 8], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629076 1459', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629076 1459', 'correlation_id': '2018-04-05 17:48:57.629076 1459', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '48265'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '48372'), ('function', 'dequeue_task'), ('message_id', '48372'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '48372'), ('message_id', '2018-04-05 17:49:01.440518 1699'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629118 1461', 'function': 'tasks.add', 'args': (10, 10), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629118 1461', 'task_id': '2018-04-05 17:48:57.629118 1461'}, 89)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '48372'), ('message_id', '2018-04-05 17:49:01.440518 1699'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629118 1461', 'function': 'tasks.add', 'args': (10, 10), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629118 1461', 'task_id': '2018-04-05 17:48:57.629118 1461'}, 89)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '47999'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 14, 'function': 'tasks.add', 'args': [7, 7], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629056 1458', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629056 1458', 'correlation_id': '2018-04-05 17:48:57.629056 1458', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '47999'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '48104'), ('function', 'dequeue_task'), ('message_id', '48104'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '48104'), ('message_id', '2018-04-05 17:49:01.740073 1711'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629135 1462', 'function': 'tasks.add', 'args': (11, 11), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629135 1462', 'task_id': '2018-04-05 17:48:57.629135 1462'}, 88)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '48104'), ('message_id', '2018-04-05 17:49:01.740073 1711'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629135 1462', 'function': 'tasks.add', 'args': (11, 11), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629135 1462', 'task_id': '2018-04-05 17:48:57.629135 1462'}, 88)), ('sender', 'Client_366')])
    
    
    Data received: 529 bytes
    Message:
    OrderedDict([('correlation_id', '48449'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 18, 'function': 'tasks.add', 'args': [9, 9], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629095 1460', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629095 1460', 'correlation_id': '2018-04-05 17:48:57.629095 1460', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '48449'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '48554'), ('function', 'dequeue_task'), ('message_id', '48554'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '48554'), ('message_id', '2018-04-05 17:49:01.927063 1719'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629152 1463', 'function': 'tasks.add', 'args': (12, 12), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629152 1463', 'task_id': '2018-04-05 17:48:57.629152 1463'}, 87)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '48554'), ('message_id', '2018-04-05 17:49:01.927063 1719'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629152 1463', 'function': 'tasks.add', 'args': (12, 12), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629152 1463', 'task_id': '2018-04-05 17:48:57.629152 1463'}, 87)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '49308'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 20, 'function': 'tasks.add', 'args': [10, 10], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629118 1461', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629118 1461', 'correlation_id': '2018-04-05 17:48:57.629118 1461', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '49308'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '49413'), ('function', 'dequeue_task'), ('message_id', '49413'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '49413'), ('message_id', '2018-04-05 17:49:02.258665 1733'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629168 1464', 'function': 'tasks.add', 'args': (13, 13), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629168 1464', 'task_id': '2018-04-05 17:48:57.629168 1464'}, 86)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '49413'), ('message_id', '2018-04-05 17:49:02.258665 1733'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629168 1464', 'function': 'tasks.add', 'args': (13, 13), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629168 1464', 'task_id': '2018-04-05 17:48:57.629168 1464'}, 86)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '49409'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 22, 'function': 'tasks.add', 'args': [11, 11], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629135 1462', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629135 1462', 'correlation_id': '2018-04-05 17:48:57.629135 1462', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '49409'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '49514'), ('function', 'dequeue_task'), ('message_id', '49514'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '49514'), ('message_id', '2018-04-05 17:49:02.502887 1743'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629184 1465', 'function': 'tasks.add', 'args': (14, 14), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629184 1465', 'task_id': '2018-04-05 17:48:57.629184 1465'}, 85)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '49514'), ('message_id', '2018-04-05 17:49:02.502887 1743'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629184 1465', 'function': 'tasks.add', 'args': (14, 14), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629184 1465', 'task_id': '2018-04-05 17:48:57.629184 1465'}, 85)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '49691'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 24, 'function': 'tasks.add', 'args': [12, 12], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629152 1463', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629152 1463', 'correlation_id': '2018-04-05 17:48:57.629152 1463', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '49691'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '49796'), ('function', 'dequeue_task'), ('message_id', '49796'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '49796'), ('message_id', '2018-04-05 17:49:02.760057 1754'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629200 1466', 'function': 'tasks.add', 'args': (15, 15), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629200 1466', 'task_id': '2018-04-05 17:48:57.629200 1466'}, 84)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '49796'), ('message_id', '2018-04-05 17:49:02.760057 1754'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629200 1466', 'function': 'tasks.add', 'args': (15, 15), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629200 1466', 'task_id': '2018-04-05 17:48:57.629200 1466'}, 84)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '50143'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 28, 'function': 'tasks.add', 'args': [14, 14], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629184 1465', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629184 1465', 'correlation_id': '2018-04-05 17:48:57.629184 1465', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '50143'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '50249'), ('function', 'dequeue_task'), ('message_id', '50249'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '50249'), ('message_id', '2018-04-05 17:49:03.097408 1770'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629215 1467', 'function': 'tasks.add', 'args': (16, 16), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629215 1467', 'task_id': '2018-04-05 17:48:57.629215 1467'}, 83)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '50249'), ('message_id', '2018-04-05 17:49:03.097408 1770'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629215 1467', 'function': 'tasks.add', 'args': (16, 16), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629215 1467', 'task_id': '2018-04-05 17:48:57.629215 1467'}, 83)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '50498'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 30, 'function': 'tasks.add', 'args': [15, 15], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629200 1466', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629200 1466', 'correlation_id': '2018-04-05 17:48:57.629200 1466', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '50498'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '50603'), ('function', 'dequeue_task'), ('message_id', '50603'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '50603'), ('message_id', '2018-04-05 17:49:03.388471 1783'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629240 1468', 'function': 'tasks.add', 'args': (17, 17), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629240 1468', 'task_id': '2018-04-05 17:48:57.629240 1468'}, 82)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '50603'), ('message_id', '2018-04-05 17:49:03.388471 1783'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629240 1468', 'function': 'tasks.add', 'args': (17, 17), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629240 1468', 'task_id': '2018-04-05 17:48:57.629240 1468'}, 82)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '50008'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 26, 'function': 'tasks.add', 'args': [13, 13], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629168 1464', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629168 1464', 'correlation_id': '2018-04-05 17:48:57.629168 1464', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '50008'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '50114'), ('function', 'dequeue_task'), ('message_id', '50114'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '50114'), ('message_id', '2018-04-05 17:49:03.628660 1794'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629295 1469', 'function': 'tasks.add', 'args': (18, 18), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629295 1469', 'task_id': '2018-04-05 17:48:57.629295 1469'}, 81)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '50114'), ('message_id', '2018-04-05 17:49:03.628660 1794'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629295 1469', 'function': 'tasks.add', 'args': (18, 18), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629295 1469', 'task_id': '2018-04-05 17:48:57.629295 1469'}, 81)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '50811'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 32, 'function': 'tasks.add', 'args': [16, 16], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629215 1467', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629215 1467', 'correlation_id': '2018-04-05 17:48:57.629215 1467', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '50811'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '51118'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 34, 'function': 'tasks.add', 'args': [17, 17], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629240 1468', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629240 1468', 'correlation_id': '2018-04-05 17:48:57.629240 1468', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '51118'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '51223'), ('function', 'dequeue_task'), ('message_id', '51223'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '51223'), ('message_id', '2018-04-05 17:49:04.044687 1810'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629314 1470', 'function': 'tasks.add', 'args': (19, 19), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629314 1470', 'task_id': '2018-04-05 17:48:57.629314 1470'}, 80)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '51223'), ('message_id', '2018-04-05 17:49:04.044687 1810'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629314 1470', 'function': 'tasks.add', 'args': (19, 19), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629314 1470', 'task_id': '2018-04-05 17:48:57.629314 1470'}, 80)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '50916'), ('function', 'dequeue_task'), ('message_id', '50916'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '50916'), ('message_id', '2018-04-05 17:49:04.187789 1815'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629333 1471', 'function': 'tasks.add', 'args': (20, 20), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629333 1471', 'task_id': '2018-04-05 17:48:57.629333 1471'}, 79)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '50916'), ('message_id', '2018-04-05 17:49:04.187789 1815'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629333 1471', 'function': 'tasks.add', 'args': (20, 20), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629333 1471', 'task_id': '2018-04-05 17:48:57.629333 1471'}, 79)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '51321'), ('function', 'dequeue_task'), ('message_id', '51321'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '51321'), ('message_id', '2018-04-05 17:49:04.319157 1820'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629351 1472', 'function': 'tasks.add', 'args': (21, 21), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629351 1472', 'task_id': '2018-04-05 17:48:57.629351 1472'}, 78)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '51321'), ('message_id', '2018-04-05 17:49:04.319157 1820'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629351 1472', 'function': 'tasks.add', 'args': (21, 21), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629351 1472', 'task_id': '2018-04-05 17:48:57.629351 1472'}, 78)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '51447'), ('function', 'dequeue_task'), ('message_id', '51447'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '51447'), ('message_id', '2018-04-05 17:49:04.456831 1826'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629373 1473', 'function': 'tasks.add', 'args': (22, 22), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629373 1473', 'task_id': '2018-04-05 17:48:57.629373 1473'}, 77)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '51447'), ('message_id', '2018-04-05 17:49:04.456831 1826'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629373 1473', 'function': 'tasks.add', 'args': (22, 22), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629373 1473', 'task_id': '2018-04-05 17:48:57.629373 1473'}, 77)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '51511'), ('function', 'dequeue_task'), ('message_id', '51511'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '51511'), ('message_id', '2018-04-05 17:49:04.757566 1832'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629520 1474', 'function': 'tasks.add', 'args': (23, 23), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629520 1474', 'task_id': '2018-04-05 17:48:57.629520 1474'}, 76)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '51511'), ('message_id', '2018-04-05 17:49:04.757566 1832'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629520 1474', 'function': 'tasks.add', 'args': (23, 23), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629520 1474', 'task_id': '2018-04-05 17:48:57.629520 1474'}, 76)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '51913'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 38, 'function': 'tasks.add', 'args': [19, 19], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629314 1470', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629314 1470', 'correlation_id': '2018-04-05 17:48:57.629314 1470', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '51913'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '52020'), ('function', 'dequeue_task'), ('message_id', '52020'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '52020'), ('message_id', '2018-04-05 17:49:05.031343 1843'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629543 1475', 'function': 'tasks.add', 'args': (24, 24), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629543 1475', 'task_id': '2018-04-05 17:48:57.629543 1475'}, 75)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '52020'), ('message_id', '2018-04-05 17:49:05.031343 1843'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629543 1475', 'function': 'tasks.add', 'args': (24, 24), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629543 1475', 'task_id': '2018-04-05 17:48:57.629543 1475'}, 75)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '51992'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 36, 'function': 'tasks.add', 'args': [18, 18], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629295 1469', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629295 1469', 'correlation_id': '2018-04-05 17:48:57.629295 1469', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '51992'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '52098'), ('function', 'dequeue_task'), ('message_id', '52098'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '52098'), ('message_id', '2018-04-05 17:49:05.447579 1856'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629559 1476', 'function': 'tasks.add', 'args': (25, 25), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629559 1476', 'task_id': '2018-04-05 17:48:57.629559 1476'}, 74)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '52098'), ('message_id', '2018-04-05 17:49:05.447579 1856'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629559 1476', 'function': 'tasks.add', 'args': (25, 25), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629559 1476', 'task_id': '2018-04-05 17:48:57.629559 1476'}, 74)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '51874'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 40, 'function': 'tasks.add', 'args': [20, 20], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629333 1471', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629333 1471', 'correlation_id': '2018-04-05 17:48:57.629333 1471', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '51874'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '51979'), ('function', 'dequeue_task'), ('message_id', '51979'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '51979'), ('message_id', '2018-04-05 17:49:05.649443 1865'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629616 1477', 'function': 'tasks.add', 'args': (26, 26), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629616 1477', 'task_id': '2018-04-05 17:48:57.629616 1477'}, 73)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '51979'), ('message_id', '2018-04-05 17:49:05.649443 1865'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629616 1477', 'function': 'tasks.add', 'args': (26, 26), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629616 1477', 'task_id': '2018-04-05 17:48:57.629616 1477'}, 73)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '52601'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 44, 'function': 'tasks.add', 'args': [22, 22], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629373 1473', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629373 1473', 'correlation_id': '2018-04-05 17:48:57.629373 1473', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '52601'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '52772'), ('function', 'dequeue_task'), ('message_id', '52772'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '52772'), ('message_id', '2018-04-05 17:49:05.881658 1875'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629636 1478', 'function': 'tasks.add', 'args': (27, 27), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629636 1478', 'task_id': '2018-04-05 17:48:57.629636 1478'}, 72)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '52772'), ('message_id', '2018-04-05 17:49:05.881658 1875'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629636 1478', 'function': 'tasks.add', 'args': (27, 27), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629636 1478', 'task_id': '2018-04-05 17:48:57.629636 1478'}, 72)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '52998'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 48, 'function': 'tasks.add', 'args': [24, 24], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629543 1475', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629543 1475', 'correlation_id': '2018-04-05 17:48:57.629543 1475', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '52998'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '53102'), ('function', 'dequeue_task'), ('message_id', '53102'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '53102'), ('message_id', '2018-04-05 17:49:06.173406 1887'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629652 1479', 'function': 'tasks.add', 'args': (28, 28), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629652 1479', 'task_id': '2018-04-05 17:48:57.629652 1479'}, 71)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '53102'), ('message_id', '2018-04-05 17:49:06.173406 1887'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629652 1479', 'function': 'tasks.add', 'args': (28, 28), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629652 1479', 'task_id': '2018-04-05 17:48:57.629652 1479'}, 71)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '53306'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 50, 'function': 'tasks.add', 'args': [25, 25], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629559 1476', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629559 1476', 'correlation_id': '2018-04-05 17:48:57.629559 1476', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '53306'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '53411'), ('function', 'dequeue_task'), ('message_id', '53411'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '53411'), ('message_id', '2018-04-05 17:49:06.466819 1897'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629668 1480', 'function': 'tasks.add', 'args': (29, 29), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629668 1480', 'task_id': '2018-04-05 17:48:57.629668 1480'}, 70)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '53411'), ('message_id', '2018-04-05 17:49:06.466819 1897'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629668 1480', 'function': 'tasks.add', 'args': (29, 29), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629668 1480', 'task_id': '2018-04-05 17:48:57.629668 1480'}, 70)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '53294'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 52, 'function': 'tasks.add', 'args': [26, 26], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629616 1477', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629616 1477', 'correlation_id': '2018-04-05 17:48:57.629616 1477', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '53294'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '53399'), ('function', 'dequeue_task'), ('message_id', '53399'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '53399'), ('message_id', '2018-04-05 17:49:06.739835 1908'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629689 1481', 'function': 'tasks.add', 'args': (30, 30), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629689 1481', 'task_id': '2018-04-05 17:48:57.629689 1481'}, 69)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '53399'), ('message_id', '2018-04-05 17:49:06.739835 1908'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629689 1481', 'function': 'tasks.add', 'args': (30, 30), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629689 1481', 'task_id': '2018-04-05 17:48:57.629689 1481'}, 69)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '53720'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 54, 'function': 'tasks.add', 'args': [27, 27], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629636 1478', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629636 1478', 'correlation_id': '2018-04-05 17:48:57.629636 1478', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '53720'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '53824'), ('function', 'dequeue_task'), ('message_id', '53824'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '53824'), ('message_id', '2018-04-05 17:49:07.163833 1919'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629715 1482', 'function': 'tasks.add', 'args': (31, 31), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629715 1482', 'task_id': '2018-04-05 17:48:57.629715 1482'}, 68)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '53824'), ('message_id', '2018-04-05 17:49:07.163833 1919'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629715 1482', 'function': 'tasks.add', 'args': (31, 31), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629715 1482', 'task_id': '2018-04-05 17:48:57.629715 1482'}, 68)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '54068'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 56, 'function': 'tasks.add', 'args': [28, 28], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629652 1479', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629652 1479', 'correlation_id': '2018-04-05 17:48:57.629652 1479', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '54068'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '54175'), ('function', 'dequeue_task'), ('message_id', '54175'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '54175'), ('message_id', '2018-04-05 17:49:07.579955 1929'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629732 1483', 'function': 'tasks.add', 'args': (32, 32), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629732 1483', 'task_id': '2018-04-05 17:48:57.629732 1483'}, 67)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '54175'), ('message_id', '2018-04-05 17:49:07.579955 1929'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629732 1483', 'function': 'tasks.add', 'args': (32, 32), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629732 1483', 'task_id': '2018-04-05 17:48:57.629732 1483'}, 67)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '54324'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 58, 'function': 'tasks.add', 'args': [29, 29], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629668 1480', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629668 1480', 'correlation_id': '2018-04-05 17:48:57.629668 1480', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '54324'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '54429'), ('function', 'dequeue_task'), ('message_id', '54429'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '54429'), ('message_id', '2018-04-05 17:49:07.813108 1939'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629749 1484', 'function': 'tasks.add', 'args': (33, 33), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629749 1484', 'task_id': '2018-04-05 17:48:57.629749 1484'}, 66)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '54429'), ('message_id', '2018-04-05 17:49:07.813108 1939'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629749 1484', 'function': 'tasks.add', 'args': (33, 33), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629749 1484', 'task_id': '2018-04-05 17:48:57.629749 1484'}, 66)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '54612'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 60, 'function': 'tasks.add', 'args': [30, 30], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629689 1481', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629689 1481', 'correlation_id': '2018-04-05 17:48:57.629689 1481', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '54612'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '54719'), ('function', 'dequeue_task'), ('message_id', '54719'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '54719'), ('message_id', '2018-04-05 17:49:08.065560 1950'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629769 1485', 'function': 'tasks.add', 'args': (34, 34), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629769 1485', 'task_id': '2018-04-05 17:48:57.629769 1485'}, 65)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '54719'), ('message_id', '2018-04-05 17:49:08.065560 1950'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629769 1485', 'function': 'tasks.add', 'args': (34, 34), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629769 1485', 'task_id': '2018-04-05 17:48:57.629769 1485'}, 65)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '55117'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 62, 'function': 'tasks.add', 'args': [31, 31], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629715 1482', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629715 1482', 'correlation_id': '2018-04-05 17:48:57.629715 1482', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '55117'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '55222'), ('function', 'dequeue_task'), ('message_id', '55222'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '55222'), ('message_id', '2018-04-05 17:49:08.303383 1960'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629788 1486', 'function': 'tasks.add', 'args': (35, 35), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629788 1486', 'task_id': '2018-04-05 17:48:57.629788 1486'}, 64)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '55222'), ('message_id', '2018-04-05 17:49:08.303383 1960'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629788 1486', 'function': 'tasks.add', 'args': (35, 35), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629788 1486', 'task_id': '2018-04-05 17:48:57.629788 1486'}, 64)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '55397'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 64, 'function': 'tasks.add', 'args': [32, 32], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629732 1483', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629732 1483', 'correlation_id': '2018-04-05 17:48:57.629732 1483', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '55397'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '55503'), ('function', 'dequeue_task'), ('message_id', '55503'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    Timeout: no result returned for request with id 2018-04-05 17:48:57.629351 1472
    
    
    Processed result:
    OrderedDict([('correlation_id', '55503'), ('message_id', '2018-04-05 17:49:08.669025 1971'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629812 1487', 'function': 'tasks.add', 'args': (36, 36), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629812 1487', 'task_id': '2018-04-05 17:48:57.629812 1487'}, 63)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '55503'), ('message_id', '2018-04-05 17:49:08.669025 1971'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629812 1487', 'function': 'tasks.add', 'args': (36, 36), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629812 1487', 'task_id': '2018-04-05 17:48:57.629812 1487'}, 63)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '55672'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 66, 'function': 'tasks.add', 'args': [33, 33], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629749 1484', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629749 1484', 'correlation_id': '2018-04-05 17:48:57.629749 1484', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '55672'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '55777'), ('function', 'dequeue_task'), ('message_id', '55777'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '55777'), ('message_id', '2018-04-05 17:49:09.030660 1981'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629828 1488', 'function': 'tasks.add', 'args': (37, 37), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629828 1488', 'task_id': '2018-04-05 17:48:57.629828 1488'}, 62)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '55777'), ('message_id', '2018-04-05 17:49:09.030660 1981'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629828 1488', 'function': 'tasks.add', 'args': (37, 37), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629828 1488', 'task_id': '2018-04-05 17:48:57.629828 1488'}, 62)), ('sender', 'Client_366')])
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '55854'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 68, 'function': 'tasks.add', 'args': [34, 34], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629769 1485', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629769 1485', 'correlation_id': '2018-04-05 17:48:57.629769 1485', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '55854'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '55960'), ('function', 'dequeue_task'), ('message_id', '55960'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '55960'), ('message_id', '2018-04-05 17:49:09.258353 1989'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629845 1489', 'function': 'tasks.add', 'args': (38, 38), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629845 1489', 'task_id': '2018-04-05 17:48:57.629845 1489'}, 61)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '55960'), ('message_id', '2018-04-05 17:49:09.258353 1989'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629845 1489', 'function': 'tasks.add', 'args': (38, 38), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629845 1489', 'task_id': '2018-04-05 17:48:57.629845 1489'}, 61)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '56187'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 70, 'function': 'tasks.add', 'args': [35, 35], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629788 1486', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629788 1486', 'correlation_id': '2018-04-05 17:48:57.629788 1486', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '56187'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '56291'), ('function', 'dequeue_task'), ('message_id', '56291'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '56291'), ('message_id', '2018-04-05 17:49:09.620722 1999'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629862 1490', 'function': 'tasks.add', 'args': (39, 39), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629862 1490', 'task_id': '2018-04-05 17:48:57.629862 1490'}, 60)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '56291'), ('message_id', '2018-04-05 17:49:09.620722 1999'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629862 1490', 'function': 'tasks.add', 'args': (39, 39), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629862 1490', 'task_id': '2018-04-05 17:48:57.629862 1490'}, 60)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '56588'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 72, 'function': 'tasks.add', 'args': [36, 36], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629812 1487', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629812 1487', 'correlation_id': '2018-04-05 17:48:57.629812 1487', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '56588'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '56694'), ('function', 'dequeue_task'), ('message_id', '56694'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '56694'), ('message_id', '2018-04-05 17:49:09.859491 2009'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629887 1491', 'function': 'tasks.add', 'args': (40, 40), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629887 1491', 'task_id': '2018-04-05 17:48:57.629887 1491'}, 59)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '56694'), ('message_id', '2018-04-05 17:49:09.859491 2009'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629887 1491', 'function': 'tasks.add', 'args': (40, 40), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629887 1491', 'task_id': '2018-04-05 17:48:57.629887 1491'}, 59)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '56785'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 74, 'function': 'tasks.add', 'args': [37, 37], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629828 1488', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629828 1488', 'correlation_id': '2018-04-05 17:48:57.629828 1488', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '56785'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '56890'), ('function', 'dequeue_task'), ('message_id', '56890'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '56890'), ('message_id', '2018-04-05 17:49:10.246169 2018'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629905 1492', 'function': 'tasks.add', 'args': (41, 41), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629905 1492', 'task_id': '2018-04-05 17:48:57.629905 1492'}, 58)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '56890'), ('message_id', '2018-04-05 17:49:10.246169 2018'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629905 1492', 'function': 'tasks.add', 'args': (41, 41), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629905 1492', 'task_id': '2018-04-05 17:48:57.629905 1492'}, 58)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '56878'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 76, 'function': 'tasks.add', 'args': [38, 38], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629845 1489', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629845 1489', 'correlation_id': '2018-04-05 17:48:57.629845 1489', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '56878'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '56983'), ('function', 'dequeue_task'), ('message_id', '56983'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '56983'), ('message_id', '2018-04-05 17:49:10.661459 2029'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629922 1493', 'function': 'tasks.add', 'args': (42, 42), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629922 1493', 'task_id': '2018-04-05 17:48:57.629922 1493'}, 57)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '56983'), ('message_id', '2018-04-05 17:49:10.661459 2029'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629922 1493', 'function': 'tasks.add', 'args': (42, 42), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629922 1493', 'task_id': '2018-04-05 17:48:57.629922 1493'}, 57)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '57481'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 78, 'function': 'tasks.add', 'args': [39, 39], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629862 1490', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629862 1490', 'correlation_id': '2018-04-05 17:48:57.629862 1490', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '57481'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '57586'), ('function', 'dequeue_task'), ('message_id', '57586'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '57586'), ('message_id', '2018-04-05 17:49:11.244763 2053'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629939 1494', 'function': 'tasks.add', 'args': (43, 43), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629939 1494', 'task_id': '2018-04-05 17:48:57.629939 1494'}, 56)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '57586'), ('message_id', '2018-04-05 17:49:11.244763 2053'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629939 1494', 'function': 'tasks.add', 'args': (43, 43), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629939 1494', 'task_id': '2018-04-05 17:48:57.629939 1494'}, 56)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '57628'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 80, 'function': 'tasks.add', 'args': [40, 40], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629887 1491', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629887 1491', 'correlation_id': '2018-04-05 17:48:57.629887 1491', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '57628'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '57732'), ('function', 'dequeue_task'), ('message_id', '57732'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '57732'), ('message_id', '2018-04-05 17:49:11.494907 2064'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629959 1495', 'function': 'tasks.add', 'args': (44, 44), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629959 1495', 'task_id': '2018-04-05 17:48:57.629959 1495'}, 55)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '57732'), ('message_id', '2018-04-05 17:49:11.494907 2064'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629959 1495', 'function': 'tasks.add', 'args': (44, 44), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629959 1495', 'task_id': '2018-04-05 17:48:57.629959 1495'}, 55)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '58748'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 82, 'function': 'tasks.add', 'args': [41, 41], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629905 1492', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629905 1492', 'correlation_id': '2018-04-05 17:48:57.629905 1492', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '58748'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    Timeout: no result returned for request with id 2018-04-05 17:48:57.629520 1474
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '58653'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 84, 'function': 'tasks.add', 'args': [42, 42], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629922 1493', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629922 1493', 'correlation_id': '2018-04-05 17:48:57.629922 1493', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '58653'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '58854'), ('function', 'dequeue_task'), ('message_id', '58854'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '58854'), ('message_id', '2018-04-05 17:49:11.891492 2081'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629978 1496', 'function': 'tasks.add', 'args': (45, 45), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629978 1496', 'task_id': '2018-04-05 17:48:57.629978 1496'}, 54)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '58854'), ('message_id', '2018-04-05 17:49:11.891492 2081'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629978 1496', 'function': 'tasks.add', 'args': (45, 45), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629978 1496', 'task_id': '2018-04-05 17:48:57.629978 1496'}, 54)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '58759'), ('function', 'dequeue_task'), ('message_id', '58759'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '58759'), ('message_id', '2018-04-05 17:49:12.025877 2086'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629994 1497', 'function': 'tasks.add', 'args': (46, 46), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629994 1497', 'task_id': '2018-04-05 17:48:57.629994 1497'}, 53)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '58759'), ('message_id', '2018-04-05 17:49:12.025877 2086'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.629994 1497', 'function': 'tasks.add', 'args': (46, 46), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.629994 1497', 'task_id': '2018-04-05 17:48:57.629994 1497'}, 53)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '59069'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 86, 'function': 'tasks.add', 'args': [43, 43], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629939 1494', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629939 1494', 'correlation_id': '2018-04-05 17:48:57.629939 1494', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '59069'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '59173'), ('function', 'dequeue_task'), ('message_id', '59173'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '59173'), ('message_id', '2018-04-05 17:49:12.477614 2096'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630011 1498', 'function': 'tasks.add', 'args': (47, 47), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630011 1498', 'task_id': '2018-04-05 17:48:57.630011 1498'}, 52)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '59173'), ('message_id', '2018-04-05 17:49:12.477614 2096'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630011 1498', 'function': 'tasks.add', 'args': (47, 47), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630011 1498', 'task_id': '2018-04-05 17:48:57.630011 1498'}, 52)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '59312'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 88, 'function': 'tasks.add', 'args': [44, 44], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629959 1495', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629959 1495', 'correlation_id': '2018-04-05 17:48:57.629959 1495', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '59312'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '59417'), ('function', 'dequeue_task'), ('message_id', '59417'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '59417'), ('message_id', '2018-04-05 17:49:12.765521 2106'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630028 1499', 'function': 'tasks.add', 'args': (48, 48), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630028 1499', 'task_id': '2018-04-05 17:48:57.630028 1499'}, 51)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '59417'), ('message_id', '2018-04-05 17:49:12.765521 2106'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630028 1499', 'function': 'tasks.add', 'args': (48, 48), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630028 1499', 'task_id': '2018-04-05 17:48:57.630028 1499'}, 51)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '59791'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 90, 'function': 'tasks.add', 'args': [45, 45], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629978 1496', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629978 1496', 'correlation_id': '2018-04-05 17:48:57.629978 1496', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '59791'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '59693'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 92, 'function': 'tasks.add', 'args': [46, 46], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629994 1497', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629994 1497', 'correlation_id': '2018-04-05 17:48:57.629994 1497', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '59693'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '59895'), ('function', 'dequeue_task'), ('message_id', '59895'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '59895'), ('message_id', '2018-04-05 17:49:13.191027 2122'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630044 1500', 'function': 'tasks.add', 'args': (49, 49), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630044 1500', 'task_id': '2018-04-05 17:48:57.630044 1500'}, 50)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '59895'), ('message_id', '2018-04-05 17:49:13.191027 2122'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630044 1500', 'function': 'tasks.add', 'args': (49, 49), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630044 1500', 'task_id': '2018-04-05 17:48:57.630044 1500'}, 50)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '59798'), ('function', 'dequeue_task'), ('message_id', '59798'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '59798'), ('message_id', '2018-04-05 17:49:13.330571 2128'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630063 1501', 'function': 'tasks.add', 'args': (50, 50), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630063 1501', 'task_id': '2018-04-05 17:48:57.630063 1501'}, 49)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '59798'), ('message_id', '2018-04-05 17:49:13.330571 2128'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630063 1501', 'function': 'tasks.add', 'args': (50, 50), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630063 1501', 'task_id': '2018-04-05 17:48:57.630063 1501'}, 49)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '60398'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 94, 'function': 'tasks.add', 'args': [47, 47], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630011 1498', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630011 1498', 'correlation_id': '2018-04-05 17:48:57.630011 1498', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '60398'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '60502'), ('function', 'dequeue_task'), ('message_id', '60502'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '60502'), ('message_id', '2018-04-05 17:49:13.603622 2139'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630080 1502', 'function': 'tasks.add', 'args': (51, 51), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630080 1502', 'task_id': '2018-04-05 17:48:57.630080 1502'}, 48)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '60502'), ('message_id', '2018-04-05 17:49:13.603622 2139'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630080 1502', 'function': 'tasks.add', 'args': (51, 51), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630080 1502', 'task_id': '2018-04-05 17:48:57.630080 1502'}, 48)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '60609'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 96, 'function': 'tasks.add', 'args': [48, 48], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630028 1499', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630028 1499', 'correlation_id': '2018-04-05 17:48:57.630028 1499', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '60609'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '60716'), ('function', 'dequeue_task'), ('message_id', '60716'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '60716'), ('message_id', '2018-04-05 17:49:13.840719 2149'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630097 1503', 'function': 'tasks.add', 'args': (52, 52), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630097 1503', 'task_id': '2018-04-05 17:48:57.630097 1503'}, 47)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '60716'), ('message_id', '2018-04-05 17:49:13.840719 2149'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630097 1503', 'function': 'tasks.add', 'args': (52, 52), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630097 1503', 'task_id': '2018-04-05 17:48:57.630097 1503'}, 47)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '60985'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 100, 'function': 'tasks.add', 'args': [50, 50], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630063 1501', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630063 1501', 'correlation_id': '2018-04-05 17:48:57.630063 1501', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '60985'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '61090'), ('function', 'dequeue_task'), ('message_id', '61090'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '61090'), ('message_id', '2018-04-05 17:49:14.136911 2159'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630112 1504', 'function': 'tasks.add', 'args': (53, 53), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630112 1504', 'task_id': '2018-04-05 17:48:57.630112 1504'}, 46)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '61090'), ('message_id', '2018-04-05 17:49:14.136911 2159'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630112 1504', 'function': 'tasks.add', 'args': (53, 53), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630112 1504', 'task_id': '2018-04-05 17:48:57.630112 1504'}, 46)), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '61073'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 98, 'function': 'tasks.add', 'args': [49, 49], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630044 1500', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630044 1500', 'correlation_id': '2018-04-05 17:48:57.630044 1500', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '61073'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '61178'), ('function', 'dequeue_task'), ('message_id', '61178'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '61178'), ('message_id', '2018-04-05 17:49:14.376275 2170'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630128 1505', 'function': 'tasks.add', 'args': (54, 54), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630128 1505', 'task_id': '2018-04-05 17:48:57.630128 1505'}, 45)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '61178'), ('message_id', '2018-04-05 17:49:14.376275 2170'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630128 1505', 'function': 'tasks.add', 'args': (54, 54), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630128 1505', 'task_id': '2018-04-05 17:48:57.630128 1505'}, 45)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '61478'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 102, 'function': 'tasks.add', 'args': [51, 51], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630080 1502', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630080 1502', 'correlation_id': '2018-04-05 17:48:57.630080 1502', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '61478'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '61583'), ('function', 'dequeue_task'), ('message_id', '61583'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '61583'), ('message_id', '2018-04-05 17:49:14.596865 2180'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630147 1506', 'function': 'tasks.add', 'args': (55, 55), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630147 1506', 'task_id': '2018-04-05 17:48:57.630147 1506'}, 44)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '61583'), ('message_id', '2018-04-05 17:49:14.596865 2180'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630147 1506', 'function': 'tasks.add', 'args': (55, 55), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630147 1506', 'task_id': '2018-04-05 17:48:57.630147 1506'}, 44)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '61714'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 104, 'function': 'tasks.add', 'args': [52, 52], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630097 1503', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630097 1503', 'correlation_id': '2018-04-05 17:48:57.630097 1503', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '61714'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '61819'), ('function', 'dequeue_task'), ('message_id', '61819'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '61819'), ('message_id', '2018-04-05 17:49:14.814513 2190'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630164 1507', 'function': 'tasks.add', 'args': (56, 56), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630164 1507', 'task_id': '2018-04-05 17:48:57.630164 1507'}, 43)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '61819'), ('message_id', '2018-04-05 17:49:14.814513 2190'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630164 1507', 'function': 'tasks.add', 'args': (56, 56), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630164 1507', 'task_id': '2018-04-05 17:48:57.630164 1507'}, 43)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '61932'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 106, 'function': 'tasks.add', 'args': [53, 53], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630112 1504', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630112 1504', 'correlation_id': '2018-04-05 17:48:57.630112 1504', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '61932'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '62037'), ('function', 'dequeue_task'), ('message_id', '62037'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '62037'), ('message_id', '2018-04-05 17:49:15.097305 2202'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630188 1508', 'function': 'tasks.add', 'args': (57, 57), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630188 1508', 'task_id': '2018-04-05 17:48:57.630188 1508'}, 42)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '62037'), ('message_id', '2018-04-05 17:49:15.097305 2202'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630188 1508', 'function': 'tasks.add', 'args': (57, 57), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630188 1508', 'task_id': '2018-04-05 17:48:57.630188 1508'}, 42)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '62226'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 108, 'function': 'tasks.add', 'args': [54, 54], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630128 1505', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630128 1505', 'correlation_id': '2018-04-05 17:48:57.630128 1505', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '62226'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '62331'), ('function', 'dequeue_task'), ('message_id', '62331'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '62331'), ('message_id', '2018-04-05 17:49:15.352204 2212'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630206 1509', 'function': 'tasks.add', 'args': (58, 58), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630206 1509', 'task_id': '2018-04-05 17:48:57.630206 1509'}, 41)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '62331'), ('message_id', '2018-04-05 17:49:15.352204 2212'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630206 1509', 'function': 'tasks.add', 'args': (58, 58), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630206 1509', 'task_id': '2018-04-05 17:48:57.630206 1509'}, 41)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '62422'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 110, 'function': 'tasks.add', 'args': [55, 55], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630147 1506', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630147 1506', 'correlation_id': '2018-04-05 17:48:57.630147 1506', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '62422'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '62526'), ('function', 'dequeue_task'), ('message_id', '62526'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '62526'), ('message_id', '2018-04-05 17:49:15.687836 2224'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630226 1510', 'function': 'tasks.add', 'args': (59, 59), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630226 1510', 'task_id': '2018-04-05 17:48:57.630226 1510'}, 40)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '62526'), ('message_id', '2018-04-05 17:49:15.687836 2224'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630226 1510', 'function': 'tasks.add', 'args': (59, 59), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630226 1510', 'task_id': '2018-04-05 17:48:57.630226 1510'}, 40)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '62892'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 114, 'function': 'tasks.add', 'args': [57, 57], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630188 1508', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630188 1508', 'correlation_id': '2018-04-05 17:48:57.630188 1508', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '62892'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '62997'), ('function', 'dequeue_task'), ('message_id', '62997'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '62997'), ('message_id', '2018-04-05 17:49:15.987179 2234'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630248 1511', 'function': 'tasks.add', 'args': (60, 60), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630248 1511', 'task_id': '2018-04-05 17:48:57.630248 1511'}, 39)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '62997'), ('message_id', '2018-04-05 17:49:15.987179 2234'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630248 1511', 'function': 'tasks.add', 'args': (60, 60), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630248 1511', 'task_id': '2018-04-05 17:48:57.630248 1511'}, 39)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '62672'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 112, 'function': 'tasks.add', 'args': [56, 56], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630164 1507', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630164 1507', 'correlation_id': '2018-04-05 17:48:57.630164 1507', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '62672'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '62777'), ('function', 'dequeue_task'), ('message_id', '62777'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '62777'), ('message_id', '2018-04-05 17:49:16.256524 2244'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630264 1512', 'function': 'tasks.add', 'args': (61, 61), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630264 1512', 'task_id': '2018-04-05 17:48:57.630264 1512'}, 38)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '62777'), ('message_id', '2018-04-05 17:49:16.256524 2244'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630264 1512', 'function': 'tasks.add', 'args': (61, 61), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630264 1512', 'task_id': '2018-04-05 17:48:57.630264 1512'}, 38)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '63271'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 116, 'function': 'tasks.add', 'args': [58, 58], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630206 1509', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630206 1509', 'correlation_id': '2018-04-05 17:48:57.630206 1509', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '63271'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '63376'), ('function', 'dequeue_task'), ('message_id', '63376'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '63376'), ('message_id', '2018-04-05 17:49:16.573306 2254'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630279 1513', 'function': 'tasks.add', 'args': (62, 62), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630279 1513', 'task_id': '2018-04-05 17:48:57.630279 1513'}, 37)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '63376'), ('message_id', '2018-04-05 17:49:16.573306 2254'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630279 1513', 'function': 'tasks.add', 'args': (62, 62), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630279 1513', 'task_id': '2018-04-05 17:48:57.630279 1513'}, 37)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '63600'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 118, 'function': 'tasks.add', 'args': [59, 59], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630226 1510', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630226 1510', 'correlation_id': '2018-04-05 17:48:57.630226 1510', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '63600'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '63705'), ('function', 'dequeue_task'), ('message_id', '63705'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '63705'), ('message_id', '2018-04-05 17:49:16.855821 2265'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630295 1514', 'function': 'tasks.add', 'args': (63, 63), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630295 1514', 'task_id': '2018-04-05 17:48:57.630295 1514'}, 36)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '63705'), ('message_id', '2018-04-05 17:49:16.855821 2265'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630295 1514', 'function': 'tasks.add', 'args': (63, 63), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630295 1514', 'task_id': '2018-04-05 17:48:57.630295 1514'}, 36)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '63753'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 120, 'function': 'tasks.add', 'args': [60, 60], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630248 1511', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630248 1511', 'correlation_id': '2018-04-05 17:48:57.630248 1511', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '63753'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '63859'), ('function', 'dequeue_task'), ('message_id', '63859'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '63859'), ('message_id', '2018-04-05 17:49:17.085398 2275'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630310 1515', 'function': 'tasks.add', 'args': (64, 64), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630310 1515', 'task_id': '2018-04-05 17:48:57.630310 1515'}, 35)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '63859'), ('message_id', '2018-04-05 17:49:17.085398 2275'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630310 1515', 'function': 'tasks.add', 'args': (64, 64), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630310 1515', 'task_id': '2018-04-05 17:48:57.630310 1515'}, 35)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '64190'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 122, 'function': 'tasks.add', 'args': [61, 61], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630264 1512', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630264 1512', 'correlation_id': '2018-04-05 17:48:57.630264 1512', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '64190'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '64297'), ('function', 'dequeue_task'), ('message_id', '64297'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '64297'), ('message_id', '2018-04-05 17:49:17.379475 2285'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630328 1516', 'function': 'tasks.add', 'args': (65, 65), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630328 1516', 'task_id': '2018-04-05 17:48:57.630328 1516'}, 34)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '64297'), ('message_id', '2018-04-05 17:49:17.379475 2285'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630328 1516', 'function': 'tasks.add', 'args': (65, 65), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630328 1516', 'task_id': '2018-04-05 17:48:57.630328 1516'}, 34)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '64466'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 124, 'function': 'tasks.add', 'args': [62, 62], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630279 1513', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630279 1513', 'correlation_id': '2018-04-05 17:48:57.630279 1513', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '64466'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '64571'), ('function', 'dequeue_task'), ('message_id', '64571'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '64571'), ('message_id', '2018-04-05 17:49:17.659626 2295'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630344 1517', 'function': 'tasks.add', 'args': (66, 66), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630344 1517', 'task_id': '2018-04-05 17:48:57.630344 1517'}, 33)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '64571'), ('message_id', '2018-04-05 17:49:17.659626 2295'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630344 1517', 'function': 'tasks.add', 'args': (66, 66), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630344 1517', 'task_id': '2018-04-05 17:48:57.630344 1517'}, 33)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '64717'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 126, 'function': 'tasks.add', 'args': [63, 63], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630295 1514', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630295 1514', 'correlation_id': '2018-04-05 17:48:57.630295 1514', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '64717'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '64824'), ('function', 'dequeue_task'), ('message_id', '64824'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '64824'), ('message_id', '2018-04-05 17:49:18.038899 2305'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630359 1518', 'function': 'tasks.add', 'args': (67, 67), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630359 1518', 'task_id': '2018-04-05 17:48:57.630359 1518'}, 32)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '64824'), ('message_id', '2018-04-05 17:49:18.038899 2305'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630359 1518', 'function': 'tasks.add', 'args': (67, 67), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630359 1518', 'task_id': '2018-04-05 17:48:57.630359 1518'}, 32)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '64932'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 128, 'function': 'tasks.add', 'args': [64, 64], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630310 1515', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630310 1515', 'correlation_id': '2018-04-05 17:48:57.630310 1515', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '64932'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '65036'), ('function', 'dequeue_task'), ('message_id', '65036'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '65036'), ('message_id', '2018-04-05 17:49:18.349767 2318'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630374 1519', 'function': 'tasks.add', 'args': (68, 68), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630374 1519', 'task_id': '2018-04-05 17:48:57.630374 1519'}, 31)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '65036'), ('message_id', '2018-04-05 17:49:18.349767 2318'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630374 1519', 'function': 'tasks.add', 'args': (68, 68), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630374 1519', 'task_id': '2018-04-05 17:48:57.630374 1519'}, 31)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '65273'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 130, 'function': 'tasks.add', 'args': [65, 65], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630328 1516', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630328 1516', 'correlation_id': '2018-04-05 17:48:57.630328 1516', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '65273'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '65377'), ('function', 'dequeue_task'), ('message_id', '65377'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '65377'), ('message_id', '2018-04-05 17:49:18.587463 2328'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630390 1520', 'function': 'tasks.add', 'args': (69, 69), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630390 1520', 'task_id': '2018-04-05 17:48:57.630390 1520'}, 30)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '65377'), ('message_id', '2018-04-05 17:49:18.587463 2328'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630390 1520', 'function': 'tasks.add', 'args': (69, 69), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630390 1520', 'task_id': '2018-04-05 17:48:57.630390 1520'}, 30)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '65628'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 132, 'function': 'tasks.add', 'args': [66, 66], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630344 1517', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630344 1517', 'correlation_id': '2018-04-05 17:48:57.630344 1517', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '65628'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '65733'), ('function', 'dequeue_task'), ('message_id', '65733'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '65733'), ('message_id', '2018-04-05 17:49:18.820101 2337'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630413 1521', 'function': 'tasks.add', 'args': (70, 70), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630413 1521', 'task_id': '2018-04-05 17:48:57.630413 1521'}, 29)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '65733'), ('message_id', '2018-04-05 17:49:18.820101 2337'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630413 1521', 'function': 'tasks.add', 'args': (70, 70), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630413 1521', 'task_id': '2018-04-05 17:48:57.630413 1521'}, 29)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '65916'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 134, 'function': 'tasks.add', 'args': [67, 67], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630359 1518', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630359 1518', 'correlation_id': '2018-04-05 17:48:57.630359 1518', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '65916'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '66021'), ('function', 'dequeue_task'), ('message_id', '66021'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '66021'), ('message_id', '2018-04-05 17:49:19.175594 2347'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630431 1522', 'function': 'tasks.add', 'args': (71, 71), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630431 1522', 'task_id': '2018-04-05 17:48:57.630431 1522'}, 28)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '66021'), ('message_id', '2018-04-05 17:49:19.175594 2347'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630431 1522', 'function': 'tasks.add', 'args': (71, 71), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630431 1522', 'task_id': '2018-04-05 17:48:57.630431 1522'}, 28)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '66103'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 136, 'function': 'tasks.add', 'args': [68, 68], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630374 1519', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630374 1519', 'correlation_id': '2018-04-05 17:48:57.630374 1519', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '66103'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '66208'), ('function', 'dequeue_task'), ('message_id', '66208'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '66208'), ('message_id', '2018-04-05 17:49:19.480364 2360'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630451 1523', 'function': 'tasks.add', 'args': (72, 72), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630451 1523', 'task_id': '2018-04-05 17:48:57.630451 1523'}, 27)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '66208'), ('message_id', '2018-04-05 17:49:19.480364 2360'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630451 1523', 'function': 'tasks.add', 'args': (72, 72), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630451 1523', 'task_id': '2018-04-05 17:48:57.630451 1523'}, 27)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '66952'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 138, 'function': 'tasks.add', 'args': [69, 69], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630390 1520', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630390 1520', 'correlation_id': '2018-04-05 17:48:57.630390 1520', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '66952'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '67059'), ('function', 'dequeue_task'), ('message_id', '67059'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '67059'), ('message_id', '2018-04-05 17:49:20.431354 2403'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630468 1524', 'function': 'tasks.add', 'args': (73, 73), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630468 1524', 'task_id': '2018-04-05 17:48:57.630468 1524'}, 26)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '67059'), ('message_id', '2018-04-05 17:49:20.431354 2403'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630468 1524', 'function': 'tasks.add', 'args': (73, 73), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630468 1524', 'task_id': '2018-04-05 17:48:57.630468 1524'}, 26)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '66711'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 140, 'function': 'tasks.add', 'args': [70, 70], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630413 1521', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630413 1521', 'correlation_id': '2018-04-05 17:48:57.630413 1521', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '66711'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '66817'), ('function', 'dequeue_task'), ('message_id', '66817'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '66817'), ('message_id', '2018-04-05 17:49:20.726090 2415'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630485 1525', 'function': 'tasks.add', 'args': (74, 74), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630485 1525', 'task_id': '2018-04-05 17:48:57.630485 1525'}, 25)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '66817'), ('message_id', '2018-04-05 17:49:20.726090 2415'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630485 1525', 'function': 'tasks.add', 'args': (74, 74), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630485 1525', 'task_id': '2018-04-05 17:48:57.630485 1525'}, 25)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '67041'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 142, 'function': 'tasks.add', 'args': [71, 71], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630431 1522', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630431 1522', 'correlation_id': '2018-04-05 17:48:57.630431 1522', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '67041'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '67146'), ('function', 'dequeue_task'), ('message_id', '67146'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '67146'), ('message_id', '2018-04-05 17:49:20.972029 2425'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630501 1526', 'function': 'tasks.add', 'args': (75, 75), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630501 1526', 'task_id': '2018-04-05 17:48:57.630501 1526'}, 24)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '67146'), ('message_id', '2018-04-05 17:49:20.972029 2425'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630501 1526', 'function': 'tasks.add', 'args': (75, 75), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630501 1526', 'task_id': '2018-04-05 17:48:57.630501 1526'}, 24)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '67643'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 144, 'function': 'tasks.add', 'args': [72, 72], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630451 1523', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630451 1523', 'correlation_id': '2018-04-05 17:48:57.630451 1523', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '67643'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '67748'), ('function', 'dequeue_task'), ('message_id', '67748'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '67748'), ('message_id', '2018-04-05 17:49:21.195062 2435'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630519 1527', 'function': 'tasks.add', 'args': (76, 76), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630519 1527', 'task_id': '2018-04-05 17:48:57.630519 1527'}, 23)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '67748'), ('message_id', '2018-04-05 17:49:21.195062 2435'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630519 1527', 'function': 'tasks.add', 'args': (76, 76), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630519 1527', 'task_id': '2018-04-05 17:48:57.630519 1527'}, 23)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '68781'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 150, 'function': 'tasks.add', 'args': [75, 75], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630501 1526', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630501 1526', 'correlation_id': '2018-04-05 17:48:57.630501 1526', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '68781'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '68886'), ('function', 'dequeue_task'), ('message_id', '68886'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '68886'), ('message_id', '2018-04-05 17:49:21.772250 2458'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630537 1528', 'function': 'tasks.add', 'args': (77, 77), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630537 1528', 'task_id': '2018-04-05 17:48:57.630537 1528'}, 22)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '68886'), ('message_id', '2018-04-05 17:49:21.772250 2458'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630537 1528', 'function': 'tasks.add', 'args': (77, 77), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630537 1528', 'task_id': '2018-04-05 17:48:57.630537 1528'}, 22)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '68891'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 152, 'function': 'tasks.add', 'args': [76, 76], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630519 1527', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630519 1527', 'correlation_id': '2018-04-05 17:48:57.630519 1527', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '68891'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '68627'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 148, 'function': 'tasks.add', 'args': [74, 74], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630485 1525', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630485 1525', 'correlation_id': '2018-04-05 17:48:57.630485 1525', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '68627'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '68996'), ('function', 'dequeue_task'), ('message_id', '68996'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '68996'), ('message_id', '2018-04-05 17:49:22.131888 2473'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630554 1529', 'function': 'tasks.add', 'args': (78, 78), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630554 1529', 'task_id': '2018-04-05 17:48:57.630554 1529'}, 21)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '68996'), ('message_id', '2018-04-05 17:49:22.131888 2473'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630554 1529', 'function': 'tasks.add', 'args': (78, 78), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630554 1529', 'task_id': '2018-04-05 17:48:57.630554 1529'}, 21)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '68735'), ('function', 'dequeue_task'), ('message_id', '68735'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '68735'), ('message_id', '2018-04-05 17:49:22.241757 2478'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630569 1530', 'function': 'tasks.add', 'args': (79, 79), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630569 1530', 'task_id': '2018-04-05 17:48:57.630569 1530'}, 20)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '68735'), ('message_id', '2018-04-05 17:49:22.241757 2478'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630569 1530', 'function': 'tasks.add', 'args': (79, 79), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630569 1530', 'task_id': '2018-04-05 17:48:57.630569 1530'}, 20)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '69410'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 146, 'function': 'tasks.add', 'args': [73, 73], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630468 1524', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630468 1524', 'correlation_id': '2018-04-05 17:48:57.630468 1524', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '69410'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '69517'), ('function', 'dequeue_task'), ('message_id', '69517'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '69517'), ('message_id', '2018-04-05 17:49:22.539772 2491'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630588 1531', 'function': 'tasks.add', 'args': (80, 80), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630588 1531', 'task_id': '2018-04-05 17:48:57.630588 1531'}, 19)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '69517'), ('message_id', '2018-04-05 17:49:22.539772 2491'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630588 1531', 'function': 'tasks.add', 'args': (80, 80), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630588 1531', 'task_id': '2018-04-05 17:48:57.630588 1531'}, 19)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '69601'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 154, 'function': 'tasks.add', 'args': [77, 77], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630537 1528', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630537 1528', 'correlation_id': '2018-04-05 17:48:57.630537 1528', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '69601'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '69706'), ('function', 'dequeue_task'), ('message_id', '69706'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '69706'), ('message_id', '2018-04-05 17:49:22.805299 2502'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630605 1532', 'function': 'tasks.add', 'args': (81, 81), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630605 1532', 'task_id': '2018-04-05 17:48:57.630605 1532'}, 18)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '69706'), ('message_id', '2018-04-05 17:49:22.805299 2502'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630605 1532', 'function': 'tasks.add', 'args': (81, 81), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630605 1532', 'task_id': '2018-04-05 17:48:57.630605 1532'}, 18)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '70001'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 158, 'function': 'tasks.add', 'args': [79, 79], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630569 1530', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630569 1530', 'correlation_id': '2018-04-05 17:48:57.630569 1530', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '70001'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '69899'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 156, 'function': 'tasks.add', 'args': [78, 78], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630554 1529', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630554 1529', 'correlation_id': '2018-04-05 17:48:57.630554 1529', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '69899'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '70106'), ('function', 'dequeue_task'), ('message_id', '70106'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '70106'), ('message_id', '2018-04-05 17:49:23.249272 2520'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630621 1533', 'function': 'tasks.add', 'args': (82, 82), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630621 1533', 'task_id': '2018-04-05 17:48:57.630621 1533'}, 17)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '70106'), ('message_id', '2018-04-05 17:49:23.249272 2520'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630621 1533', 'function': 'tasks.add', 'args': (82, 82), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630621 1533', 'task_id': '2018-04-05 17:48:57.630621 1533'}, 17)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '70005'), ('function', 'dequeue_task'), ('message_id', '70005'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '70005'), ('message_id', '2018-04-05 17:49:23.363389 2525'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630640 1534', 'function': 'tasks.add', 'args': (83, 83), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630640 1534', 'task_id': '2018-04-05 17:48:57.630640 1534'}, 16)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '70005'), ('message_id', '2018-04-05 17:49:23.363389 2525'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630640 1534', 'function': 'tasks.add', 'args': (83, 83), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630640 1534', 'task_id': '2018-04-05 17:48:57.630640 1534'}, 16)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '70639'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 162, 'function': 'tasks.add', 'args': [81, 81], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630605 1532', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630605 1532', 'correlation_id': '2018-04-05 17:48:57.630605 1532', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '70639'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '70744'), ('function', 'dequeue_task'), ('message_id', '70744'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '70744'), ('message_id', '2018-04-05 17:49:23.595202 2535'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630656 1535', 'function': 'tasks.add', 'args': (84, 84), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630656 1535', 'task_id': '2018-04-05 17:48:57.630656 1535'}, 15)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '70744'), ('message_id', '2018-04-05 17:49:23.595202 2535'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630656 1535', 'function': 'tasks.add', 'args': (84, 84), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630656 1535', 'task_id': '2018-04-05 17:48:57.630656 1535'}, 15)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '71014'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 166, 'function': 'tasks.add', 'args': [83, 83], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630640 1534', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630640 1534', 'correlation_id': '2018-04-05 17:48:57.630640 1534', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '71014'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '71119'), ('function', 'dequeue_task'), ('message_id', '71119'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '71119'), ('message_id', '2018-04-05 17:49:24.143923 2553'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630672 1536', 'function': 'tasks.add', 'args': (85, 85), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630672 1536', 'task_id': '2018-04-05 17:48:57.630672 1536'}, 14)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '71119'), ('message_id', '2018-04-05 17:49:24.143923 2553'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630672 1536', 'function': 'tasks.add', 'args': (85, 85), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630672 1536', 'task_id': '2018-04-05 17:48:57.630672 1536'}, 14)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '71109'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 164, 'function': 'tasks.add', 'args': [82, 82], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630621 1533', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630621 1533', 'correlation_id': '2018-04-05 17:48:57.630621 1533', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '71109'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '71214'), ('function', 'dequeue_task'), ('message_id', '71214'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '71214'), ('message_id', '2018-04-05 17:49:24.487515 2563'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630691 1537', 'function': 'tasks.add', 'args': (86, 86), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630691 1537', 'task_id': '2018-04-05 17:48:57.630691 1537'}, 13)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '71214'), ('message_id', '2018-04-05 17:49:24.487515 2563'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630691 1537', 'function': 'tasks.add', 'args': (86, 86), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630691 1537', 'task_id': '2018-04-05 17:48:57.630691 1537'}, 13)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '71480'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 168, 'function': 'tasks.add', 'args': [84, 84], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630656 1535', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630656 1535', 'correlation_id': '2018-04-05 17:48:57.630656 1535', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '71480'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '71587'), ('function', 'dequeue_task'), ('message_id', '71587'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '71587'), ('message_id', '2018-04-05 17:49:24.819773 2574'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630707 1538', 'function': 'tasks.add', 'args': (87, 87), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630707 1538', 'task_id': '2018-04-05 17:48:57.630707 1538'}, 12)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '71587'), ('message_id', '2018-04-05 17:49:24.819773 2574'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630707 1538', 'function': 'tasks.add', 'args': (87, 87), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630707 1538', 'task_id': '2018-04-05 17:48:57.630707 1538'}, 12)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '72431'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 172, 'function': 'tasks.add', 'args': [86, 86], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630691 1537', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630691 1537', 'correlation_id': '2018-04-05 17:48:57.630691 1537', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '72431'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '72538'), ('function', 'dequeue_task'), ('message_id', '72538'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '72538'), ('message_id', '2018-04-05 17:49:25.477342 2599'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630723 1539', 'function': 'tasks.add', 'args': (88, 88), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630723 1539', 'task_id': '2018-04-05 17:48:57.630723 1539'}, 11)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '72538'), ('message_id', '2018-04-05 17:49:25.477342 2599'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630723 1539', 'function': 'tasks.add', 'args': (88, 88), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630723 1539', 'task_id': '2018-04-05 17:48:57.630723 1539'}, 11)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '72411'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 170, 'function': 'tasks.add', 'args': [85, 85], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630672 1536', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630672 1536', 'correlation_id': '2018-04-05 17:48:57.630672 1536', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '72411'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '72515'), ('function', 'dequeue_task'), ('message_id', '72515'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '72515'), ('message_id', '2018-04-05 17:49:25.794499 2609'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630740 1540', 'function': 'tasks.add', 'args': (89, 89), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630740 1540', 'task_id': '2018-04-05 17:48:57.630740 1540'}, 10)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '72515'), ('message_id', '2018-04-05 17:49:25.794499 2609'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630740 1540', 'function': 'tasks.add', 'args': (89, 89), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630740 1540', 'task_id': '2018-04-05 17:48:57.630740 1540'}, 10)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '72608'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 160, 'function': 'tasks.add', 'args': [80, 80], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630588 1531', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630588 1531', 'correlation_id': '2018-04-05 17:48:57.630588 1531', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '72608'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '72607'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 174, 'function': 'tasks.add', 'args': [87, 87], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630707 1538', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630707 1538', 'correlation_id': '2018-04-05 17:48:57.630707 1538', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '72607'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '72713'), ('function', 'dequeue_task'), ('message_id', '72713'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '72713'), ('message_id', '2018-04-05 17:49:26.221595 2625'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630758 1541', 'function': 'tasks.add', 'args': (90, 90), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630758 1541', 'task_id': '2018-04-05 17:48:57.630758 1541'}, 9)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '72713'), ('message_id', '2018-04-05 17:49:26.221595 2625'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630758 1541', 'function': 'tasks.add', 'args': (90, 90), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630758 1541', 'task_id': '2018-04-05 17:48:57.630758 1541'}, 9)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '72714'), ('function', 'dequeue_task'), ('message_id', '72714'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '72714'), ('message_id', '2018-04-05 17:49:26.354117 2631'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630777 1542', 'function': 'tasks.add', 'args': (91, 91), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630777 1542', 'task_id': '2018-04-05 17:48:57.630777 1542'}, 8)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '72714'), ('message_id', '2018-04-05 17:49:26.354117 2631'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630777 1542', 'function': 'tasks.add', 'args': (91, 91), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630777 1542', 'task_id': '2018-04-05 17:48:57.630777 1542'}, 8)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '73427'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 176, 'function': 'tasks.add', 'args': [88, 88], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630723 1539', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630723 1539', 'correlation_id': '2018-04-05 17:48:57.630723 1539', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '73427'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '73531'), ('function', 'dequeue_task'), ('message_id', '73531'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '73531'), ('message_id', '2018-04-05 17:49:26.615059 2641'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630793 1543', 'function': 'tasks.add', 'args': (92, 92), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630793 1543', 'task_id': '2018-04-05 17:48:57.630793 1543'}, 7)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '73531'), ('message_id', '2018-04-05 17:49:26.615059 2641'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630793 1543', 'function': 'tasks.add', 'args': (92, 92), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630793 1543', 'task_id': '2018-04-05 17:48:57.630793 1543'}, 7)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '73575'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 178, 'function': 'tasks.add', 'args': [89, 89], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630740 1540', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630740 1540', 'correlation_id': '2018-04-05 17:48:57.630740 1540', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '73575'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '73680'), ('function', 'dequeue_task'), ('message_id', '73680'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '73680'), ('message_id', '2018-04-05 17:49:27.213530 2669'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630809 1544', 'function': 'tasks.add', 'args': (93, 93), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630809 1544', 'task_id': '2018-04-05 17:48:57.630809 1544'}, 6)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '73680'), ('message_id', '2018-04-05 17:49:27.213530 2669'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630809 1544', 'function': 'tasks.add', 'args': (93, 93), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630809 1544', 'task_id': '2018-04-05 17:48:57.630809 1544'}, 6)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '74386'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 184, 'function': 'tasks.add', 'args': [92, 92], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630793 1543', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630793 1543', 'correlation_id': '2018-04-05 17:48:57.630793 1543', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '74386'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '74492'), ('function', 'dequeue_task'), ('message_id', '74492'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '74492'), ('message_id', '2018-04-05 17:49:27.485799 2680'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630825 1545', 'function': 'tasks.add', 'args': (94, 94), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630825 1545', 'task_id': '2018-04-05 17:48:57.630825 1545'}, 5)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '74492'), ('message_id', '2018-04-05 17:49:27.485799 2680'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630825 1545', 'function': 'tasks.add', 'args': (94, 94), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630825 1545', 'task_id': '2018-04-05 17:48:57.630825 1545'}, 5)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '74080'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 182, 'function': 'tasks.add', 'args': [91, 91], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630777 1542', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630777 1542', 'correlation_id': '2018-04-05 17:48:57.630777 1542', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '74080'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '74185'), ('function', 'dequeue_task'), ('message_id', '74185'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '74185'), ('message_id', '2018-04-05 17:49:27.706257 2690'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630847 1546', 'function': 'tasks.add', 'args': (95, 95), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630847 1546', 'task_id': '2018-04-05 17:48:57.630847 1546'}, 4)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '74185'), ('message_id', '2018-04-05 17:49:27.706257 2690'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630847 1546', 'function': 'tasks.add', 'args': (95, 95), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630847 1546', 'task_id': '2018-04-05 17:48:57.630847 1546'}, 4)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '75016'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 186, 'function': 'tasks.add', 'args': [93, 93], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630809 1544', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630809 1544', 'correlation_id': '2018-04-05 17:48:57.630809 1544', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '75016'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '75121'), ('function', 'dequeue_task'), ('message_id', '75121'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '75121'), ('message_id', '2018-04-05 17:49:28.055895 2706'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630867 1547', 'function': 'tasks.add', 'args': (96, 96), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630867 1547', 'task_id': '2018-04-05 17:48:57.630867 1547'}, 3)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '75121'), ('message_id', '2018-04-05 17:49:28.055895 2706'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630867 1547', 'function': 'tasks.add', 'args': (96, 96), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630867 1547', 'task_id': '2018-04-05 17:48:57.630867 1547'}, 3)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '75328'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 188, 'function': 'tasks.add', 'args': [94, 94], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630825 1545', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630825 1545', 'correlation_id': '2018-04-05 17:48:57.630825 1545', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '75328'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '75433'), ('function', 'dequeue_task'), ('message_id', '75433'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '75433'), ('message_id', '2018-04-05 17:49:28.286776 2716'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630883 1548', 'function': 'tasks.add', 'args': (97, 97), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630883 1548', 'task_id': '2018-04-05 17:48:57.630883 1548'}, 2)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '75433'), ('message_id', '2018-04-05 17:49:28.286776 2716'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630883 1548', 'function': 'tasks.add', 'args': (97, 97), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630883 1548', 'task_id': '2018-04-05 17:48:57.630883 1548'}, 2)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '75507'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 190, 'function': 'tasks.add', 'args': [95, 95], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630847 1546', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630847 1546', 'correlation_id': '2018-04-05 17:48:57.630847 1546', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '75507'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '75612'), ('function', 'dequeue_task'), ('message_id', '75612'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '75612'), ('message_id', '2018-04-05 17:49:28.544529 2727'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630899 1549', 'function': 'tasks.add', 'args': (98, 98), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630899 1549', 'task_id': '2018-04-05 17:48:57.630899 1549'}, 1)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '75612'), ('message_id', '2018-04-05 17:49:28.544529 2727'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630899 1549', 'function': 'tasks.add', 'args': (98, 98), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630899 1549', 'task_id': '2018-04-05 17:48:57.630899 1549'}, 1)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '75856'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 192, 'function': 'tasks.add', 'args': [96, 96], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630867 1547', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630867 1547', 'correlation_id': '2018-04-05 17:48:57.630867 1547', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '75856'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '75961'), ('function', 'dequeue_task'), ('message_id', '75961'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '75961'), ('message_id', '2018-04-05 17:49:28.802810 2739'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630915 1550', 'function': 'tasks.add', 'args': (99, 99), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630915 1550', 'task_id': '2018-04-05 17:48:57.630915 1550'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 495 bytes
    Message:
    OrderedDict([('correlation_id', '75961'), ('message_id', '2018-04-05 17:49:28.802810 2739'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:48:57.630915 1550', 'function': 'tasks.add', 'args': (99, 99), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:48:57.630915 1550', 'task_id': '2018-04-05 17:48:57.630915 1550'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '76009'), ('function', 'dequeue_task'), ('message_id', '76009'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '76009'), ('message_id', '2018-04-05 17:49:28.911442 2744'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '76009'), ('message_id', '2018-04-05 17:49:28.911442 2744'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '76171'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 194, 'function': 'tasks.add', 'args': [97, 97], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630883 1548', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630883 1548', 'correlation_id': '2018-04-05 17:48:57.630883 1548', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '76171'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '76274'), ('function', 'dequeue_task'), ('message_id', '76274'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '76274'), ('message_id', '2018-04-05 17:49:29.173118 2755'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '76274'), ('message_id', '2018-04-05 17:49:29.173118 2755'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '76244'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 196, 'function': 'tasks.add', 'args': [98, 98], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630899 1549', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630899 1549', 'correlation_id': '2018-04-05 17:48:57.630899 1549', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '76244'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '76348'), ('function', 'dequeue_task'), ('message_id', '76348'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '76348'), ('message_id', '2018-04-05 17:49:29.380059 2764'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 209 bytes
    Message:
    OrderedDict([('correlation_id', '76348'), ('message_id', '2018-04-05 17:49:29.380059 2764'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 532 bytes
    Message:
    OrderedDict([('correlation_id', '76616'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 198, 'function': 'tasks.add', 'args': [99, 99], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.630915 1550', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.630915 1550', 'correlation_id': '2018-04-05 17:48:57.630915 1550', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '76616'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    Timeout: no result returned for request with id 2018-04-05 17:48:57.630758 1541
    




    [[0, 2, 4, 6, 8, 10, 12, 14, 16, 18],
     [20, 22, 24, 26, 28, 30, 32, 34, 36, 38],
     [40, None, 44, None, 48, 50, 52, 54, 56, 58],
     [60, 62, 64, 66, 68, 70, 72, 74, 76, 78],
     [80, 82, 84, 86, 88, 90, 92, 94, 96, 98],
     [100, 102, 104, 106, 108, 110, 112, 114, 116, 118],
     [120, 122, 124, 126, 128, 130, 132, 134, 136, 138],
     [140, 142, 144, 146, 148, 150, 152, 154, 156, 158],
     [160, 162, 164, 166, 168, 170, 172, 174, 176, 178],
     [None, 182, 184, 186, 188, 190, 192, 194, 196, 198]]



### Word Count
最後我們以 Hadoop 領域中的 "Hello World" 範例 "Word Count" 來測試。  

我們會把一個文字檔的內容拆解成 words 並將每個 word 發送給 workers 處理，workers 要做的主要是一個`mapper`處理：
```
def mapper(word):
    return (word, 1) if len(word) > 3 else None
```
worker 會將處理的結果傳回來，client 這邊會有一個`reduce`function 將結果彙整。  

我們可以在 ESP32 cluster 上面這樣做：


```python
import word_count

text_file = os.path.join('..', '..', 'codes', 'broccoli', 'client', 'test.txt')
words_count, counts = word_count.count_words(text_file)
print('********** result:\nwords count: {}\n\n{}\n**********'.format(words_count, counts[:10]))
```

    
    Sending 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:49:30.141453 2806'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:49:30.141453 2806'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 261 bytes
    Message:
    OrderedDict([('correlation_id', '2018-04-05 17:49:30.141453 2806'), ('function', 'fetch_task'), ('kwargs', {'broker': 'Client_366'}), ('message_id', '2018-04-05 17:49:30.141453 2806'), ('message_type', 'function'), ('receiver', 'Hub'), ('reply_to', 'Client_366'), ('sender', 'Client_366')])
    
    
    Data received: 531 bytes
    Message:
    OrderedDict([('correlation_id', '77132'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': 42, 'function': 'tasks.add', 'args': [21, 21], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:48:57.629351 1472', 'message_type': 'result', 'task_id': '2018-04-05 17:48:57.629351 1472', 'correlation_id': '2018-04-05 17:48:57.629351 1472', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '77132'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '77240'), ('function', 'dequeue_task'), ('message_id', '77240'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '77240'), ('message_id', '2018-04-05 17:49:30.581714 2915'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.141388 2805', 'function': 'tasks.mapper', 'args': ("Aesop's",), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.141388 2805', 'task_id': '2018-04-05 17:49:30.141388 2805'}, 93)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '77240'), ('message_id', '2018-04-05 17:49:30.581714 2915'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.141388 2805', 'function': 'tasks.mapper', 'args': ("Aesop's",), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.141388 2805', 'task_id': '2018-04-05 17:49:30.141388 2805'}, 93)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '77838'), ('function', 'dequeue_task'), ('message_id', '77838'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '77838'), ('message_id', '2018-04-05 17:49:30.730985 2920'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195857 2808', 'function': 'tasks.mapper', 'args': ('Fables',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195857 2808', 'task_id': '2018-04-05 17:49:30.195857 2808'}, 92)), ('sender', 'Client_366')])
    
    
    Sending 501 bytes
    Message:
    OrderedDict([('correlation_id', '77838'), ('message_id', '2018-04-05 17:49:30.730985 2920'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195857 2808', 'function': 'tasks.mapper', 'args': ('Fables',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195857 2808', 'task_id': '2018-04-05 17:49:30.195857 2808'}, 92)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '78143'), ('function', 'dequeue_task'), ('message_id', '78143'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '78143'), ('message_id', '2018-04-05 17:49:30.918084 2927'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195914 2809', 'function': 'tasks.mapper', 'args': ('Translated',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195914 2809', 'task_id': '2018-04-05 17:49:30.195914 2809'}, 91)), ('sender', 'Client_366')])
    
    
    Sending 505 bytes
    Message:
    OrderedDict([('correlation_id', '78143'), ('message_id', '2018-04-05 17:49:30.918084 2927'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195914 2809', 'function': 'tasks.mapper', 'args': ('Translated',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195914 2809', 'task_id': '2018-04-05 17:49:30.195914 2809'}, 91)), ('sender', 'Client_366')])
    
    
    Data received: 547 bytes
    Message:
    OrderedDict([('correlation_id', '78550'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Fables', 1], 'function': 'tasks.mapper', 'args': ['Fables'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.195857 2808', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.195857 2808', 'correlation_id': '2018-04-05 17:49:30.195857 2808', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '78550'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '78658'), ('function', 'dequeue_task'), ('message_id', '78658'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '78658'), ('message_id', '2018-04-05 17:49:31.506592 2955'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195938 2810', 'function': 'tasks.mapper', 'args': ('by',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195938 2810', 'task_id': '2018-04-05 17:49:30.195938 2810'}, 90)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '78658'), ('message_id', '2018-04-05 17:49:31.506592 2955'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195938 2810', 'function': 'tasks.mapper', 'args': ('by',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195938 2810', 'task_id': '2018-04-05 17:49:30.195938 2810'}, 90)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '78603'), ('function', 'dequeue_task'), ('message_id', '78603'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '78603'), ('message_id', '2018-04-05 17:49:31.635579 2961'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195959 2811', 'function': 'tasks.mapper', 'args': ('George',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195959 2811', 'task_id': '2018-04-05 17:49:30.195959 2811'}, 89)), ('sender', 'Client_366')])
    
    
    Sending 501 bytes
    Message:
    OrderedDict([('correlation_id', '78603'), ('message_id', '2018-04-05 17:49:31.635579 2961'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195959 2811', 'function': 'tasks.mapper', 'args': ('George',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195959 2811', 'task_id': '2018-04-05 17:49:30.195959 2811'}, 89)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '78644'), ('function', 'dequeue_task'), ('message_id', '78644'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '78644'), ('message_id', '2018-04-05 17:49:31.784822 2966'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195978 2812', 'function': 'tasks.mapper', 'args': ('Fyler',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195978 2812', 'task_id': '2018-04-05 17:49:30.195978 2812'}, 88)), ('sender', 'Client_366')])
    
    
    Sending 500 bytes
    Message:
    OrderedDict([('correlation_id', '78644'), ('message_id', '2018-04-05 17:49:31.784822 2966'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195978 2812', 'function': 'tasks.mapper', 'args': ('Fyler',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195978 2812', 'task_id': '2018-04-05 17:49:30.195978 2812'}, 88)), ('sender', 'Client_366')])
    
    
    Data received: 555 bytes
    Message:
    OrderedDict([('correlation_id', '78658'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Translated', 1], 'function': 'tasks.mapper', 'args': ['Translated'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.195914 2809', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.195914 2809', 'correlation_id': '2018-04-05 17:49:30.195914 2809', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '78658'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '78767'), ('function', 'dequeue_task'), ('message_id', '78767'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '78767'), ('message_id', '2018-04-05 17:49:32.022259 2975'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195994 2813', 'function': 'tasks.mapper', 'args': ('Townsend',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195994 2813', 'task_id': '2018-04-05 17:49:30.195994 2813'}, 87)), ('sender', 'Client_366')])
    
    
    Sending 503 bytes
    Message:
    OrderedDict([('correlation_id', '78767'), ('message_id', '2018-04-05 17:49:32.022259 2975'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.195994 2813', 'function': 'tasks.mapper', 'args': ('Townsend',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.195994 2813', 'task_id': '2018-04-05 17:49:30.195994 2813'}, 87)), ('sender', 'Client_366')])
    
    
    Data received: 534 bytes
    Message:
    OrderedDict([('correlation_id', '79390'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['by'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.195938 2810', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.195938 2810', 'correlation_id': '2018-04-05 17:49:30.195938 2810', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '79390'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '79497'), ('function', 'dequeue_task'), ('message_id', '79497'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '79497'), ('message_id', '2018-04-05 17:49:32.290540 2987'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196011 2814', 'function': 'tasks.mapper', 'args': ('The',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196011 2814', 'task_id': '2018-04-05 17:49:30.196011 2814'}, 86)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '79497'), ('message_id', '2018-04-05 17:49:32.290540 2987'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196011 2814', 'function': 'tasks.mapper', 'args': ('The',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196011 2814', 'task_id': '2018-04-05 17:49:30.196011 2814'}, 86)), ('sender', 'Client_366')])
    
    
    Data received: 547 bytes
    Message:
    OrderedDict([('correlation_id', '79405'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['George', 1], 'function': 'tasks.mapper', 'args': ['George'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.195959 2811', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.195959 2811', 'correlation_id': '2018-04-05 17:49:30.195959 2811', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '79405'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '79513'), ('function', 'dequeue_task'), ('message_id', '79513'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '79513'), ('message_id', '2018-04-05 17:49:32.502547 2996'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196026 2815', 'function': 'tasks.mapper', 'args': ('Wolf',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196026 2815', 'task_id': '2018-04-05 17:49:30.196026 2815'}, 85)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '79513'), ('message_id', '2018-04-05 17:49:32.502547 2996'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196026 2815', 'function': 'tasks.mapper', 'args': ('Wolf',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196026 2815', 'task_id': '2018-04-05 17:49:30.196026 2815'}, 85)), ('sender', 'Client_366')])
    
    
    Data received: 545 bytes
    Message:
    OrderedDict([('correlation_id', '79553'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Fyler', 1], 'function': 'tasks.mapper', 'args': ['Fyler'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.195978 2812', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.195978 2812', 'correlation_id': '2018-04-05 17:49:30.195978 2812', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '79553'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '79663'), ('function', 'dequeue_task'), ('message_id', '79663'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '79663'), ('message_id', '2018-04-05 17:49:32.765328 3006'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196046 2816', 'function': 'tasks.mapper', 'args': ('and',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196046 2816', 'task_id': '2018-04-05 17:49:30.196046 2816'}, 84)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '79663'), ('message_id', '2018-04-05 17:49:32.765328 3006'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196046 2816', 'function': 'tasks.mapper', 'args': ('and',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196046 2816', 'task_id': '2018-04-05 17:49:30.196046 2816'}, 84)), ('sender', 'Client_366')])
    
    
    Data received: 551 bytes
    Message:
    OrderedDict([('correlation_id', '79717'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Townsend', 1], 'function': 'tasks.mapper', 'args': ['Townsend'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.195994 2813', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.195994 2813', 'correlation_id': '2018-04-05 17:49:30.195994 2813', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '79717'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '79826'), ('function', 'dequeue_task'), ('message_id', '79826'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '79826'), ('message_id', '2018-04-05 17:49:32.963716 3015'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196062 2817', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196062 2817', 'task_id': '2018-04-05 17:49:30.196062 2817'}, 83)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '79826'), ('message_id', '2018-04-05 17:49:32.963716 3015'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196062 2817', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196062 2817', 'task_id': '2018-04-05 17:49:30.196062 2817'}, 83)), ('sender', 'Client_366')])
    
    Timeout: no result returned for request with id 2018-04-05 17:49:30.141388 2805
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '80487'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['and'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196046 2816', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196046 2816', 'correlation_id': '2018-04-05 17:49:30.196046 2816', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '80487'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '80594'), ('function', 'dequeue_task'), ('message_id', '80594'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '80594'), ('message_id', '2018-04-05 17:49:33.683050 3049'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196078 2818', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196078 2818', 'task_id': '2018-04-05 17:49:30.196078 2818'}, 82)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '80594'), ('message_id', '2018-04-05 17:49:33.683050 3049'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196078 2818', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196078 2818', 'task_id': '2018-04-05 17:49:30.196078 2818'}, 82)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '80693'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['the'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196062 2817', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196062 2817', 'correlation_id': '2018-04-05 17:49:30.196062 2817', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '80693'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '80798'), ('function', 'dequeue_task'), ('message_id', '80798'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '80798'), ('message_id', '2018-04-05 17:49:33.928239 3059'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196094 2819', 'function': 'tasks.mapper', 'args': ('WOLF',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196094 2819', 'task_id': '2018-04-05 17:49:30.196094 2819'}, 81)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '80798'), ('message_id', '2018-04-05 17:49:33.928239 3059'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196094 2819', 'function': 'tasks.mapper', 'args': ('WOLF',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196094 2819', 'task_id': '2018-04-05 17:49:30.196094 2819'}, 81)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '80151'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['The'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196011 2814', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196011 2814', 'correlation_id': '2018-04-05 17:49:30.196011 2814', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '80151'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '80257'), ('function', 'dequeue_task'), ('message_id', '80257'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '80257'), ('message_id', '2018-04-05 17:49:34.231488 3072'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196110 2820', 'function': 'tasks.mapper', 'args': ('meeting',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196110 2820', 'task_id': '2018-04-05 17:49:30.196110 2820'}, 80)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '80257'), ('message_id', '2018-04-05 17:49:34.231488 3072'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196110 2820', 'function': 'tasks.mapper', 'args': ('meeting',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196110 2820', 'task_id': '2018-04-05 17:49:30.196110 2820'}, 80)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '80878'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Wolf', 1], 'function': 'tasks.mapper', 'args': ['Wolf'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196026 2815', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196026 2815', 'correlation_id': '2018-04-05 17:49:30.196026 2815', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '80878'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '80987'), ('function', 'dequeue_task'), ('message_id', '80987'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '80987'), ('message_id', '2018-04-05 17:49:34.480654 3083'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196126 2821', 'function': 'tasks.mapper', 'args': ('with',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196126 2821', 'task_id': '2018-04-05 17:49:30.196126 2821'}, 79)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '80987'), ('message_id', '2018-04-05 17:49:34.480654 3083'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196126 2821', 'function': 'tasks.mapper', 'args': ('with',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196126 2821', 'task_id': '2018-04-05 17:49:30.196126 2821'}, 79)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '81780'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['WOLF', 1], 'function': 'tasks.mapper', 'args': ['WOLF'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196094 2819', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196094 2819', 'correlation_id': '2018-04-05 17:49:30.196094 2819', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '81780'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '81844'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Lamb', 1], 'function': 'tasks.mapper', 'args': ['Lamb'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196078 2818', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196078 2818', 'correlation_id': '2018-04-05 17:49:30.196078 2818', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '81844'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '81888'), ('function', 'dequeue_task'), ('message_id', '81888'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '81888'), ('message_id', '2018-04-05 17:49:34.895806 3102'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196141 2822', 'function': 'tasks.mapper', 'args': ('a',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196141 2822', 'task_id': '2018-04-05 17:49:30.196141 2822'}, 78)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '81888'), ('message_id', '2018-04-05 17:49:34.895806 3102'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196141 2822', 'function': 'tasks.mapper', 'args': ('a',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196141 2822', 'task_id': '2018-04-05 17:49:30.196141 2822'}, 78)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '81951'), ('function', 'dequeue_task'), ('message_id', '81951'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '81951'), ('message_id', '2018-04-05 17:49:35.002740 3107'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196157 2823', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196157 2823', 'task_id': '2018-04-05 17:49:30.196157 2823'}, 77)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '81951'), ('message_id', '2018-04-05 17:49:35.002740 3107'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196157 2823', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196157 2823', 'task_id': '2018-04-05 17:49:30.196157 2823'}, 77)), ('sender', 'Client_366')])
    
    
    Data received: 549 bytes
    Message:
    OrderedDict([('correlation_id', '82150'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['meeting', 1], 'function': 'tasks.mapper', 'args': ['meeting'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196110 2820', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196110 2820', 'correlation_id': '2018-04-05 17:49:30.196110 2820', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '82150'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '82260'), ('function', 'dequeue_task'), ('message_id', '82260'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '82260'), ('message_id', '2018-04-05 17:49:35.287659 3120'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196178 2824', 'function': 'tasks.mapper', 'args': ('astray',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196178 2824', 'task_id': '2018-04-05 17:49:30.196178 2824'}, 76)), ('sender', 'Client_366')])
    
    
    Sending 501 bytes
    Message:
    OrderedDict([('correlation_id', '82260'), ('message_id', '2018-04-05 17:49:35.287659 3120'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196178 2824', 'function': 'tasks.mapper', 'args': ('astray',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196178 2824', 'task_id': '2018-04-05 17:49:30.196178 2824'}, 76)), ('sender', 'Client_366')])
    
    
    Data received: 533 bytes
    Message:
    OrderedDict([('correlation_id', '82695'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['a'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196141 2822', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196141 2822', 'correlation_id': '2018-04-05 17:49:30.196141 2822', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '82695'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '82800'), ('function', 'dequeue_task'), ('message_id', '82800'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '82800'), ('message_id', '2018-04-05 17:49:35.621901 3136'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196195 2825', 'function': 'tasks.mapper', 'args': ('from',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196195 2825', 'task_id': '2018-04-05 17:49:30.196195 2825'}, 75)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '82800'), ('message_id', '2018-04-05 17:49:35.621901 3136'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196195 2825', 'function': 'tasks.mapper', 'args': ('from',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196195 2825', 'task_id': '2018-04-05 17:49:30.196195 2825'}, 75)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '82761'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Lamb', 1], 'function': 'tasks.mapper', 'args': ['Lamb'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196157 2823', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196157 2823', 'correlation_id': '2018-04-05 17:49:30.196157 2823', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '82761'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '82869'), ('function', 'dequeue_task'), ('message_id', '82869'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '82869'), ('message_id', '2018-04-05 17:49:35.845512 3145'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196211 2826', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196211 2826', 'task_id': '2018-04-05 17:49:30.196211 2826'}, 74)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '82869'), ('message_id', '2018-04-05 17:49:35.845512 3145'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196211 2826', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196211 2826', 'task_id': '2018-04-05 17:49:30.196211 2826'}, 74)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '83314'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['with', 1], 'function': 'tasks.mapper', 'args': ['with'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196126 2821', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196126 2821', 'correlation_id': '2018-04-05 17:49:30.196126 2821', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '83314'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 547 bytes
    Message:
    OrderedDict([('correlation_id', '83069'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['astray', 1], 'function': 'tasks.mapper', 'args': ['astray'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196178 2824', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196178 2824', 'correlation_id': '2018-04-05 17:49:30.196178 2824', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '83069'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '83421'), ('function', 'dequeue_task'), ('message_id', '83421'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '83421'), ('message_id', '2018-04-05 17:49:36.323809 3165'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196229 2827', 'function': 'tasks.mapper', 'args': ('fold',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196229 2827', 'task_id': '2018-04-05 17:49:30.196229 2827'}, 73)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '83421'), ('message_id', '2018-04-05 17:49:36.323809 3165'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196229 2827', 'function': 'tasks.mapper', 'args': ('fold',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196229 2827', 'task_id': '2018-04-05 17:49:30.196229 2827'}, 73)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '83179'), ('function', 'dequeue_task'), ('message_id', '83179'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '83179'), ('message_id', '2018-04-05 17:49:36.441146 3170'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196244 2828', 'function': 'tasks.mapper', 'args': ('resolved',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196244 2828', 'task_id': '2018-04-05 17:49:30.196244 2828'}, 72)), ('sender', 'Client_366')])
    
    
    Sending 503 bytes
    Message:
    OrderedDict([('correlation_id', '83179'), ('message_id', '2018-04-05 17:49:36.441146 3170'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196244 2828', 'function': 'tasks.mapper', 'args': ('resolved',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196244 2828', 'task_id': '2018-04-05 17:49:30.196244 2828'}, 72)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '83417'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['from', 1], 'function': 'tasks.mapper', 'args': ['from'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196195 2825', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196195 2825', 'correlation_id': '2018-04-05 17:49:30.196195 2825', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '83417'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '83524'), ('function', 'dequeue_task'), ('message_id', '83524'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '83524'), ('message_id', '2018-04-05 17:49:36.697974 3181'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196261 2829', 'function': 'tasks.mapper', 'args': ('not',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196261 2829', 'task_id': '2018-04-05 17:49:30.196261 2829'}, 71)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '83524'), ('message_id', '2018-04-05 17:49:36.697974 3181'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196261 2829', 'function': 'tasks.mapper', 'args': ('not',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196261 2829', 'task_id': '2018-04-05 17:49:30.196261 2829'}, 71)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '83557'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['the'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196211 2826', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196211 2826', 'correlation_id': '2018-04-05 17:49:30.196211 2826', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '83557'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '83664'), ('function', 'dequeue_task'), ('message_id', '83664'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '83664'), ('message_id', '2018-04-05 17:49:36.979501 3193'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196277 2830', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196277 2830', 'task_id': '2018-04-05 17:49:30.196277 2830'}, 70)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '83664'), ('message_id', '2018-04-05 17:49:36.979501 3193'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196277 2830', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196277 2830', 'task_id': '2018-04-05 17:49:30.196277 2830'}, 70)), ('sender', 'Client_366')])
    
    
    Data received: 551 bytes
    Message:
    OrderedDict([('correlation_id', '84276'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['resolved', 1], 'function': 'tasks.mapper', 'args': ['resolved'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196244 2828', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196244 2828', 'correlation_id': '2018-04-05 17:49:30.196244 2828', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '84276'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '84236'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['fold', 1], 'function': 'tasks.mapper', 'args': ['fold'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196229 2827', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196229 2827', 'correlation_id': '2018-04-05 17:49:30.196229 2827', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '84236'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '84384'), ('function', 'dequeue_task'), ('message_id', '84384'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '84384'), ('message_id', '2018-04-05 17:49:37.409209 3211'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196293 2831', 'function': 'tasks.mapper', 'args': ('lay',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196293 2831', 'task_id': '2018-04-05 17:49:30.196293 2831'}, 69)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '84384'), ('message_id', '2018-04-05 17:49:37.409209 3211'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196293 2831', 'function': 'tasks.mapper', 'args': ('lay',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196293 2831', 'task_id': '2018-04-05 17:49:30.196293 2831'}, 69)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '84343'), ('function', 'dequeue_task'), ('message_id', '84343'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '84343'), ('message_id', '2018-04-05 17:49:37.542000 3216'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196308 2832', 'function': 'tasks.mapper', 'args': ('violent',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196308 2832', 'task_id': '2018-04-05 17:49:30.196308 2832'}, 68)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '84343'), ('message_id', '2018-04-05 17:49:37.542000 3216'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196308 2832', 'function': 'tasks.mapper', 'args': ('violent',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196308 2832', 'task_id': '2018-04-05 17:49:30.196308 2832'}, 68)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '84980'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['not'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196261 2829', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196261 2829', 'correlation_id': '2018-04-05 17:49:30.196261 2829', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '84980'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '85088'), ('function', 'dequeue_task'), ('message_id', '85088'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '85088'), ('message_id', '2018-04-05 17:49:37.951372 3236'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196326 2833', 'function': 'tasks.mapper', 'args': ('hands',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196326 2833', 'task_id': '2018-04-05 17:49:30.196326 2833'}, 67)), ('sender', 'Client_366')])
    
    
    Sending 500 bytes
    Message:
    OrderedDict([('correlation_id', '85088'), ('message_id', '2018-04-05 17:49:37.951372 3236'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196326 2833', 'function': 'tasks.mapper', 'args': ('hands',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196326 2833', 'task_id': '2018-04-05 17:49:30.196326 2833'}, 67)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '85310'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['lay'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196293 2831', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196293 2831', 'correlation_id': '2018-04-05 17:49:30.196293 2831', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '85310'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '85415'), ('function', 'dequeue_task'), ('message_id', '85415'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '85415'), ('message_id', '2018-04-05 17:49:38.238634 3247'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196343 2834', 'function': 'tasks.mapper', 'args': ('on',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196343 2834', 'task_id': '2018-04-05 17:49:30.196343 2834'}, 66)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '85415'), ('message_id', '2018-04-05 17:49:38.238634 3247'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196343 2834', 'function': 'tasks.mapper', 'args': ('on',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196343 2834', 'task_id': '2018-04-05 17:49:30.196343 2834'}, 66)), ('sender', 'Client_366')])
    
    
    Data received: 549 bytes
    Message:
    OrderedDict([('correlation_id', '85275'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['violent', 1], 'function': 'tasks.mapper', 'args': ['violent'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196308 2832', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196308 2832', 'correlation_id': '2018-04-05 17:49:30.196308 2832', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '85275'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '85384'), ('function', 'dequeue_task'), ('message_id', '85384'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '85384'), ('message_id', '2018-04-05 17:49:38.457005 3257'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196359 2835', 'function': 'tasks.mapper', 'args': ('him',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196359 2835', 'task_id': '2018-04-05 17:49:30.196359 2835'}, 65)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '85384'), ('message_id', '2018-04-05 17:49:38.457005 3257'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196359 2835', 'function': 'tasks.mapper', 'args': ('him',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196359 2835', 'task_id': '2018-04-05 17:49:30.196359 2835'}, 65)), ('sender', 'Client_366')])
    
    
    Data received: 545 bytes
    Message:
    OrderedDict([('correlation_id', '85777'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['hands', 1], 'function': 'tasks.mapper', 'args': ['hands'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196326 2833', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196326 2833', 'correlation_id': '2018-04-05 17:49:30.196326 2833', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '85777'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '85884'), ('function', 'dequeue_task'), ('message_id', '85884'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '85884'), ('message_id', '2018-04-05 17:49:38.732374 3269'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196374 2836', 'function': 'tasks.mapper', 'args': ('but',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196374 2836', 'task_id': '2018-04-05 17:49:30.196374 2836'}, 64)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '85884'), ('message_id', '2018-04-05 17:49:38.732374 3269'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196374 2836', 'function': 'tasks.mapper', 'args': ('but',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196374 2836', 'task_id': '2018-04-05 17:49:30.196374 2836'}, 64)), ('sender', 'Client_366')])
    
    
    Data received: 534 bytes
    Message:
    OrderedDict([('correlation_id', '86088'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['on'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196343 2834', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196343 2834', 'correlation_id': '2018-04-05 17:49:30.196343 2834', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '86088'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '86193'), ('function', 'dequeue_task'), ('message_id', '86193'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '86193'), ('message_id', '2018-04-05 17:49:39.037000 3283'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196390 2837', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196390 2837', 'task_id': '2018-04-05 17:49:30.196390 2837'}, 63)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Sending 497 bytes
    
    Message:
    OrderedDict([('correlation_id', '86193'), ('message_id', '2018-04-05 17:49:39.037000 3283'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196390 2837', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196390 2837', 'task_id': '2018-04-05 17:49:30.196390 2837'}, 63)), ('sender', 'Client_366')])
    
    Message:
    OrderedDict([('correlation_id', '86194'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['him'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196359 2835', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196359 2835', 'correlation_id': '2018-04-05 17:49:30.196359 2835', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '86194'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '86299'), ('function', 'dequeue_task'), ('message_id', '86299'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '86299'), ('message_id', '2018-04-05 17:49:39.277678 3293'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196414 2838', 'function': 'tasks.mapper', 'args': ('find',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196414 2838', 'task_id': '2018-04-05 17:49:30.196414 2838'}, 62)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '86299'), ('message_id', '2018-04-05 17:49:39.277678 3293'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196414 2838', 'function': 'tasks.mapper', 'args': ('find',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196414 2838', 'task_id': '2018-04-05 17:49:30.196414 2838'}, 62)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '86414'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['but'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196374 2836', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196374 2836', 'correlation_id': '2018-04-05 17:49:30.196374 2836', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '86414'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '86521'), ('function', 'dequeue_task'), ('message_id', '86521'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '86521'), ('message_id', '2018-04-05 17:49:39.526873 3304'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196431 2839', 'function': 'tasks.mapper', 'args': ('some',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196431 2839', 'task_id': '2018-04-05 17:49:30.196431 2839'}, 61)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '86521'), ('message_id', '2018-04-05 17:49:39.526873 3304'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196431 2839', 'function': 'tasks.mapper', 'args': ('some',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196431 2839', 'task_id': '2018-04-05 17:49:30.196431 2839'}, 61)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '87009'), ('function', 'dequeue_task'), ('message_id', '87009'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '87009'), ('message_id', '2018-04-05 17:49:39.701754 3312'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196448 2840', 'function': 'tasks.mapper', 'args': ('plea',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196448 2840', 'task_id': '2018-04-05 17:49:30.196448 2840'}, 60)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '87009'), ('message_id', '2018-04-05 17:49:39.701754 3312'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196448 2840', 'function': 'tasks.mapper', 'args': ('plea',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196448 2840', 'task_id': '2018-04-05 17:49:30.196448 2840'}, 60)), ('sender', 'Client_366')])
    
    
    Data received: 534 bytes
    Message:
    OrderedDict([('correlation_id', '87031'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['to'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196390 2837', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196390 2837', 'correlation_id': '2018-04-05 17:49:30.196390 2837', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '87031'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '87136'), ('function', 'dequeue_task'), ('message_id', '87136'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '87136'), ('message_id', '2018-04-05 17:49:39.997098 3324'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196463 2841', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196463 2841', 'task_id': '2018-04-05 17:49:30.196463 2841'}, 59)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '87136'), ('message_id', '2018-04-05 17:49:39.997098 3324'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196463 2841', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196463 2841', 'task_id': '2018-04-05 17:49:30.196463 2841'}, 59)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '87224'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['some', 1], 'function': 'tasks.mapper', 'args': ['some'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196431 2839', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196431 2839', 'correlation_id': '2018-04-05 17:49:30.196431 2839', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '87224'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '87331'), ('function', 'dequeue_task'), ('message_id', '87331'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '87331'), ('message_id', '2018-04-05 17:49:40.268519 3336'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196480 2842', 'function': 'tasks.mapper', 'args': ('justify',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196480 2842', 'task_id': '2018-04-05 17:49:30.196480 2842'}, 58)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '87331'), ('message_id', '2018-04-05 17:49:40.268519 3336'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196480 2842', 'function': 'tasks.mapper', 'args': ('justify',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196480 2842', 'task_id': '2018-04-05 17:49:30.196480 2842'}, 58)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '87032'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['find', 1], 'function': 'tasks.mapper', 'args': ['find'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196414 2838', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196414 2838', 'correlation_id': '2018-04-05 17:49:30.196414 2838', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '87032'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '87141'), ('function', 'dequeue_task'), ('message_id', '87141'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '87141'), ('message_id', '2018-04-05 17:49:40.474799 3344'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196506 2843', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196506 2843', 'task_id': '2018-04-05 17:49:30.196506 2843'}, 57)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '87141'), ('message_id', '2018-04-05 17:49:40.474799 3344'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196506 2843', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196506 2843', 'task_id': '2018-04-05 17:49:30.196506 2843'}, 57)), ('sender', 'Client_366')])
    
    
    Data received: 534 bytes
    Message:
    OrderedDict([('correlation_id', '87909'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['to'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196463 2841', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196463 2841', 'correlation_id': '2018-04-05 17:49:30.196463 2841', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '87909'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '88015'), ('function', 'dequeue_task'), ('message_id', '88015'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '88015'), ('message_id', '2018-04-05 17:49:40.759542 3356'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196523 2844', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196523 2844', 'task_id': '2018-04-05 17:49:30.196523 2844'}, 56)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '88015'), ('message_id', '2018-04-05 17:49:40.759542 3356'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196523 2844', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196523 2844', 'task_id': '2018-04-05 17:49:30.196523 2844'}, 56)), ('sender', 'Client_366')])
    
    Timeout: no result returned for request with id 2018-04-05 17:49:30.196277 2830
    
    Data received: 534 bytes
    Message:
    OrderedDict([('correlation_id', '88353'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['to'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196506 2843', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196506 2843', 'correlation_id': '2018-04-05 17:49:30.196506 2843', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '88353'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '87999'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['plea', 1], 'function': 'tasks.mapper', 'args': ['plea'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196448 2840', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196448 2840', 'correlation_id': '2018-04-05 17:49:30.196448 2840', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '87999'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '88457'), ('function', 'dequeue_task'), ('message_id', '88457'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '88457'), ('message_id', '2018-04-05 17:49:41.418632 3388'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196540 2845', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196540 2845', 'task_id': '2018-04-05 17:49:30.196540 2845'}, 55)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '88457'), ('message_id', '2018-04-05 17:49:41.418632 3388'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196540 2845', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196540 2845', 'task_id': '2018-04-05 17:49:30.196540 2845'}, 55)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '88107'), ('function', 'dequeue_task'), ('message_id', '88107'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '88107'), ('message_id', '2018-04-05 17:49:41.565641 3394'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196555 2846', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196555 2846', 'task_id': '2018-04-05 17:49:30.196555 2846'}, 54)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '88107'), ('message_id', '2018-04-05 17:49:41.565641 3394'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196555 2846', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196555 2846', 'task_id': '2018-04-05 17:49:30.196555 2846'}, 54)), ('sender', 'Client_366')])
    
    
    Data received: 549 bytes
    Message:
    OrderedDict([('correlation_id', '88335'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['justify', 1], 'function': 'tasks.mapper', 'args': ['justify'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196480 2842', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196480 2842', 'correlation_id': '2018-04-05 17:49:30.196480 2842', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '88335'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '88443'), ('function', 'dequeue_task'), ('message_id', '88443'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '88443'), ('message_id', '2018-04-05 17:49:41.835262 3405'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196571 2847', 'function': 'tasks.mapper', 'args': ("Wolf's",), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196571 2847', 'task_id': '2018-04-05 17:49:30.196571 2847'}, 53)), ('sender', 'Client_366')])
    
    
    Sending 501 bytes
    Message:
    OrderedDict([('correlation_id', '88443'), ('message_id', '2018-04-05 17:49:41.835262 3405'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196571 2847', 'function': 'tasks.mapper', 'args': ("Wolf's",), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196571 2847', 'task_id': '2018-04-05 17:49:30.196571 2847'}, 53)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '88587'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['the'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196523 2844', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196523 2844', 'correlation_id': '2018-04-05 17:49:30.196523 2844', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '88587'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '88692'), ('function', 'dequeue_task'), ('message_id', '88692'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '88692'), ('message_id', '2018-04-05 17:49:42.097486 3416'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196588 2848', 'function': 'tasks.mapper', 'args': ('right',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196588 2848', 'task_id': '2018-04-05 17:49:30.196588 2848'}, 52)), ('sender', 'Client_366')])
    
    
    Sending 500 bytes
    Message:
    OrderedDict([('correlation_id', '88692'), ('message_id', '2018-04-05 17:49:42.097486 3416'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196588 2848', 'function': 'tasks.mapper', 'args': ('right',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196588 2848', 'task_id': '2018-04-05 17:49:30.196588 2848'}, 52)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '89279'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Lamb', 1], 'function': 'tasks.mapper', 'args': ['Lamb'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196540 2845', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196540 2845', 'correlation_id': '2018-04-05 17:49:30.196540 2845', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '89279'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '89286'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['the'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196555 2846', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196555 2846', 'correlation_id': '2018-04-05 17:49:30.196555 2846', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '89286'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '89388'), ('function', 'dequeue_task'), ('message_id', '89388'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '89388'), ('message_id', '2018-04-05 17:49:42.593738 3431'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196604 2849', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196604 2849', 'task_id': '2018-04-05 17:49:30.196604 2849'}, 51)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '89388'), ('message_id', '2018-04-05 17:49:42.593738 3431'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196604 2849', 'function': 'tasks.mapper', 'args': ('to',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196604 2849', 'task_id': '2018-04-05 17:49:30.196604 2849'}, 51)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '89393'), ('function', 'dequeue_task'), ('message_id', '89393'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '89393'), ('message_id', '2018-04-05 17:49:42.781010 3436'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196621 2850', 'function': 'tasks.mapper', 'args': ('eat',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196621 2850', 'task_id': '2018-04-05 17:49:30.196621 2850'}, 50)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '89393'), ('message_id', '2018-04-05 17:49:42.781010 3436'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196621 2850', 'function': 'tasks.mapper', 'args': ('eat',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196621 2850', 'task_id': '2018-04-05 17:49:30.196621 2850'}, 50)), ('sender', 'Client_366')])
    
    
    Data received: 547 bytes
    Message:
    OrderedDict([('correlation_id', '89597'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ["Wolf's", 1], 'function': 'tasks.mapper', 'args': ["Wolf's"], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196571 2847', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196571 2847', 'correlation_id': '2018-04-05 17:49:30.196571 2847', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '89597'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '89705'), ('function', 'dequeue_task'), ('message_id', '89705'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '89705'), ('message_id', '2018-04-05 17:49:43.126149 3446'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196641 2851', 'function': 'tasks.mapper', 'args': ('him',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196641 2851', 'task_id': '2018-04-05 17:49:30.196641 2851'}, 49)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '89705'), ('message_id', '2018-04-05 17:49:43.126149 3446'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196641 2851', 'function': 'tasks.mapper', 'args': ('him',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196641 2851', 'task_id': '2018-04-05 17:49:30.196641 2851'}, 49)), ('sender', 'Client_366')])
    
    
    Data received: 545 bytes
    Message:
    OrderedDict([('correlation_id', '90032'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['right', 1], 'function': 'tasks.mapper', 'args': ['right'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196588 2848', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196588 2848', 'correlation_id': '2018-04-05 17:49:30.196588 2848', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '90032'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '90141'), ('function', 'dequeue_task'), ('message_id', '90141'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '90141'), ('message_id', '2018-04-05 17:49:43.444589 3456'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196657 2852', 'function': 'tasks.mapper', 'args': ('He',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196657 2852', 'task_id': '2018-04-05 17:49:30.196657 2852'}, 48)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '90141'), ('message_id', '2018-04-05 17:49:43.444589 3456'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196657 2852', 'function': 'tasks.mapper', 'args': ('He',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196657 2852', 'task_id': '2018-04-05 17:49:30.196657 2852'}, 48)), ('sender', 'Client_366')])
    
    Data received: 534 bytes
    
    Message:
    OrderedDict([('correlation_id', '90513'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['to'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196604 2849', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196604 2849', 'correlation_id': '2018-04-05 17:49:30.196604 2849', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '90513'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '90522'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['eat'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196621 2850', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196621 2850', 'correlation_id': '2018-04-05 17:49:30.196621 2850', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '90522'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '90619'), ('function', 'dequeue_task'), ('message_id', '90619'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '90619'), ('message_id', '2018-04-05 17:49:43.845200 3474'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196673 2853', 'function': 'tasks.mapper', 'args': ('thus',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196673 2853', 'task_id': '2018-04-05 17:49:30.196673 2853'}, 47)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '90619'), ('message_id', '2018-04-05 17:49:43.845200 3474'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196673 2853', 'function': 'tasks.mapper', 'args': ('thus',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196673 2853', 'task_id': '2018-04-05 17:49:30.196673 2853'}, 47)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '90628'), ('function', 'dequeue_task'), ('message_id', '90628'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '90628'), ('message_id', '2018-04-05 17:49:43.954678 3479'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196690 2854', 'function': 'tasks.mapper', 'args': ('addressed',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196690 2854', 'task_id': '2018-04-05 17:49:30.196690 2854'}, 46)), ('sender', 'Client_366')])
    
    
    Sending 504 bytes
    Message:
    OrderedDict([('correlation_id', '90628'), ('message_id', '2018-04-05 17:49:43.954678 3479'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196690 2854', 'function': 'tasks.mapper', 'args': ('addressed',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196690 2854', 'task_id': '2018-04-05 17:49:30.196690 2854'}, 46)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '90981'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['him'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196641 2851', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196641 2851', 'correlation_id': '2018-04-05 17:49:30.196641 2851', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '90981'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '91086'), ('function', 'dequeue_task'), ('message_id', '91086'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '91086'), ('message_id', '2018-04-05 17:49:44.245054 3490'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196706 2855', 'function': 'tasks.mapper', 'args': ('him:',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196706 2855', 'task_id': '2018-04-05 17:49:30.196706 2855'}, 45)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '91086'), ('message_id', '2018-04-05 17:49:44.245054 3490'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196706 2855', 'function': 'tasks.mapper', 'args': ('him:',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196706 2855', 'task_id': '2018-04-05 17:49:30.196706 2855'}, 45)), ('sender', 'Client_366')])
    
    
    Data received: 534 bytes
    Message:
    OrderedDict([('correlation_id', '91189'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['He'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196657 2852', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196657 2852', 'correlation_id': '2018-04-05 17:49:30.196657 2852', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '91189'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '91294'), ('function', 'dequeue_task'), ('message_id', '91294'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '91294'), ('message_id', '2018-04-05 17:49:44.553819 3501'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196722 2856', 'function': 'tasks.mapper', 'args': ('"Sirrah',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196722 2856', 'task_id': '2018-04-05 17:49:30.196722 2856'}, 44)), ('sender', 'Client_366')])
    
    
    Sending 503 bytes
    Message:
    OrderedDict([('correlation_id', '91294'), ('message_id', '2018-04-05 17:49:44.553819 3501'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196722 2856', 'function': 'tasks.mapper', 'args': ('"Sirrah',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196722 2856', 'task_id': '2018-04-05 17:49:30.196722 2856'}, 44)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '91683'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['thus', 1], 'function': 'tasks.mapper', 'args': ['thus'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196673 2853', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196673 2853', 'correlation_id': '2018-04-05 17:49:30.196673 2853', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '91683'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 553 bytes
    Message:
    OrderedDict([('correlation_id', '91691'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['addressed', 1], 'function': 'tasks.mapper', 'args': ['addressed'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196690 2854', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196690 2854', 'correlation_id': '2018-04-05 17:49:30.196690 2854', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '91691'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '91791'), ('function', 'dequeue_task'), ('message_id', '91791'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '91791'), ('message_id', '2018-04-05 17:49:44.931557 3516'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196743 2857', 'function': 'tasks.mapper', 'args': ('last',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196743 2857', 'task_id': '2018-04-05 17:49:30.196743 2857'}, 43)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '91791'), ('message_id', '2018-04-05 17:49:44.931557 3516'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196743 2857', 'function': 'tasks.mapper', 'args': ('last',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196743 2857', 'task_id': '2018-04-05 17:49:30.196743 2857'}, 43)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '91800'), ('function', 'dequeue_task'), ('message_id', '91800'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '91800'), ('message_id', '2018-04-05 17:49:45.040938 3521'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196760 2858', 'function': 'tasks.mapper', 'args': ('year',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196760 2858', 'task_id': '2018-04-05 17:49:30.196760 2858'}, 42)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '91800'), ('message_id', '2018-04-05 17:49:45.040938 3521'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196760 2858', 'function': 'tasks.mapper', 'args': ('year',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196760 2858', 'task_id': '2018-04-05 17:49:30.196760 2858'}, 42)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '92093'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['him:', 1], 'function': 'tasks.mapper', 'args': ['him:'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196706 2855', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196706 2855', 'correlation_id': '2018-04-05 17:49:30.196706 2855', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '92093'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '92201'), ('function', 'dequeue_task'), ('message_id', '92201'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '92201'), ('message_id', '2018-04-05 17:49:45.355045 3534'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196776 2859', 'function': 'tasks.mapper', 'args': ('you',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196776 2859', 'task_id': '2018-04-05 17:49:30.196776 2859'}, 41)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '92201'), ('message_id', '2018-04-05 17:49:45.355045 3534'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196776 2859', 'function': 'tasks.mapper', 'args': ('you',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196776 2859', 'task_id': '2018-04-05 17:49:30.196776 2859'}, 41)), ('sender', 'Client_366')])
    
    
    Data received: 551 bytes
    Message:
    OrderedDict([('correlation_id', '92470'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['"Sirrah', 1], 'function': 'tasks.mapper', 'args': ['"Sirrah'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196722 2856', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196722 2856', 'correlation_id': '2018-04-05 17:49:30.196722 2856', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '92470'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '92577'), ('function', 'dequeue_task'), ('message_id', '92577'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '92577'), ('message_id', '2018-04-05 17:49:45.629332 3547'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196793 2860', 'function': 'tasks.mapper', 'args': ('grossly',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196793 2860', 'task_id': '2018-04-05 17:49:30.196793 2860'}, 40)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '92577'), ('message_id', '2018-04-05 17:49:45.629332 3547'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196793 2860', 'function': 'tasks.mapper', 'args': ('grossly',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196793 2860', 'task_id': '2018-04-05 17:49:30.196793 2860'}, 40)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '92758'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['last', 1], 'function': 'tasks.mapper', 'args': ['last'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196743 2857', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196743 2857', 'correlation_id': '2018-04-05 17:49:30.196743 2857', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '92758'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '92770'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['year', 1], 'function': 'tasks.mapper', 'args': ['year'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196760 2858', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196760 2858', 'correlation_id': '2018-04-05 17:49:30.196760 2858', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '92770'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '92866'), ('function', 'dequeue_task'), ('message_id', '92866'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '92866'), ('message_id', '2018-04-05 17:49:46.002239 3562'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196808 2861', 'function': 'tasks.mapper', 'args': ('grossly',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196808 2861', 'task_id': '2018-04-05 17:49:30.196808 2861'}, 39)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '92866'), ('message_id', '2018-04-05 17:49:46.002239 3562'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196808 2861', 'function': 'tasks.mapper', 'args': ('grossly',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196808 2861', 'task_id': '2018-04-05 17:49:30.196808 2861'}, 39)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '92879'), ('function', 'dequeue_task'), ('message_id', '92879'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '92879'), ('message_id', '2018-04-05 17:49:46.165858 3570'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196826 2862', 'function': 'tasks.mapper', 'args': ('insulted',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196826 2862', 'task_id': '2018-04-05 17:49:30.196826 2862'}, 38)), ('sender', 'Client_366')])
    
    
    Sending 503 bytes
    Message:
    OrderedDict([('correlation_id', '92879'), ('message_id', '2018-04-05 17:49:46.165858 3570'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196826 2862', 'function': 'tasks.mapper', 'args': ('insulted',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196826 2862', 'task_id': '2018-04-05 17:49:30.196826 2862'}, 38)), ('sender', 'Client_366')])
    
    
    Data received: 549 bytes
    Message:
    OrderedDict([('correlation_id', '93477'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['grossly', 1], 'function': 'tasks.mapper', 'args': ['grossly'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196793 2860', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196793 2860', 'correlation_id': '2018-04-05 17:49:30.196793 2860', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '93477'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '93584'), ('function', 'dequeue_task'), ('message_id', '93584'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '93584'), ('message_id', '2018-04-05 17:49:46.458221 3582'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196843 2863', 'function': 'tasks.mapper', 'args': ('me"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196843 2863', 'task_id': '2018-04-05 17:49:30.196843 2863'}, 37)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '93584'), ('message_id', '2018-04-05 17:49:46.458221 3582'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196843 2863', 'function': 'tasks.mapper', 'args': ('me"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196843 2863', 'task_id': '2018-04-05 17:49:30.196843 2863'}, 37)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '93134'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['you'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196776 2859', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196776 2859', 'correlation_id': '2018-04-05 17:49:30.196776 2859', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '93134'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '93240'), ('function', 'dequeue_task'), ('message_id', '93240'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '93240'), ('message_id', '2018-04-05 17:49:46.699770 3593'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196862 2864', 'function': 'tasks.mapper', 'args': ('"Indeed"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196862 2864', 'task_id': '2018-04-05 17:49:30.196862 2864'}, 36)), ('sender', 'Client_366')])
    
    
    Sending 505 bytes
    Message:
    OrderedDict([('correlation_id', '93240'), ('message_id', '2018-04-05 17:49:46.699770 3593'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196862 2864', 'function': 'tasks.mapper', 'args': ('"Indeed"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196862 2864', 'task_id': '2018-04-05 17:49:30.196862 2864'}, 36)), ('sender', 'Client_366')])
    
    
    Data received: 549 bytes
    Message:
    OrderedDict([('correlation_id', '93874'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['grossly', 1], 'function': 'tasks.mapper', 'args': ['grossly'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196808 2861', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196808 2861', 'correlation_id': '2018-04-05 17:49:30.196808 2861', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '93874'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '93982'), ('function', 'dequeue_task'), ('message_id', '93982'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '93982'), ('message_id', '2018-04-05 17:49:47.064128 3605'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196878 2865', 'function': 'tasks.mapper', 'args': ('bleated',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196878 2865', 'task_id': '2018-04-05 17:49:30.196878 2865'}, 35)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '93982'), ('message_id', '2018-04-05 17:49:47.064128 3605'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196878 2865', 'function': 'tasks.mapper', 'args': ('bleated',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196878 2865', 'task_id': '2018-04-05 17:49:30.196878 2865'}, 35)), ('sender', 'Client_366')])
    
    
    Data received: 551 bytes
    Message:
    OrderedDict([('correlation_id', '93921'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['insulted', 1], 'function': 'tasks.mapper', 'args': ['insulted'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196826 2862', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196826 2862', 'correlation_id': '2018-04-05 17:49:30.196826 2862', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '93921'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '94029'), ('function', 'dequeue_task'), ('message_id', '94029'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '94029'), ('message_id', '2018-04-05 17:49:47.337046 3616'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196894 2866', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196894 2866', 'task_id': '2018-04-05 17:49:30.196894 2866'}, 34)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '94029'), ('message_id', '2018-04-05 17:49:47.337046 3616'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196894 2866', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196894 2866', 'task_id': '2018-04-05 17:49:30.196894 2866'}, 34)), ('sender', 'Client_366')])
    
    
    Data received: 555 bytes
    Message:
    OrderedDict([('correlation_id', '94538'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['"Indeed"', 1], 'function': 'tasks.mapper', 'args': ['"Indeed"'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196862 2864', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196862 2864', 'correlation_id': '2018-04-05 17:49:30.196862 2864', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '94538'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '94647'), ('function', 'dequeue_task'), ('message_id', '94647'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '94647'), ('message_id', '2018-04-05 17:49:47.574550 3626'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196913 2867', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196913 2867', 'task_id': '2018-04-05 17:49:30.196913 2867'}, 33)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '94647'), ('message_id', '2018-04-05 17:49:47.574550 3626'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196913 2867', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196913 2867', 'task_id': '2018-04-05 17:49:30.196913 2867'}, 33)), ('sender', 'Client_366')])
    
    
    Data received: 536 bytes
    Message:
    OrderedDict([('correlation_id', '94329'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['me"'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196843 2863', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196843 2863', 'correlation_id': '2018-04-05 17:49:30.196843 2863', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '94329'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '94434'), ('function', 'dequeue_task'), ('message_id', '94434'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '94434'), ('message_id', '2018-04-05 17:49:47.826030 3636'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196929 2868', 'function': 'tasks.mapper', 'args': ('in',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196929 2868', 'task_id': '2018-04-05 17:49:30.196929 2868'}, 32)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '94434'), ('message_id', '2018-04-05 17:49:47.826030 3636'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196929 2868', 'function': 'tasks.mapper', 'args': ('in',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196929 2868', 'task_id': '2018-04-05 17:49:30.196929 2868'}, 32)), ('sender', 'Client_366')])
    
    
    Data received: 549 bytes
    Message:
    OrderedDict([('correlation_id', '94953'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['bleated', 1], 'function': 'tasks.mapper', 'args': ['bleated'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196878 2865', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196878 2865', 'correlation_id': '2018-04-05 17:49:30.196878 2865', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '94953'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '95062'), ('function', 'dequeue_task'), ('message_id', '95062'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '95062'), ('message_id', '2018-04-05 17:49:48.102699 3647'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196947 2869', 'function': 'tasks.mapper', 'args': ('a',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196947 2869', 'task_id': '2018-04-05 17:49:30.196947 2869'}, 31)), ('sender', 'Client_366')])
    
    
    Sending 496 bytes
    Message:
    OrderedDict([('correlation_id', '95062'), ('message_id', '2018-04-05 17:49:48.102699 3647'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196947 2869', 'function': 'tasks.mapper', 'args': ('a',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196947 2869', 'task_id': '2018-04-05 17:49:30.196947 2869'}, 31)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '95159'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['the'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196894 2866', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196894 2866', 'correlation_id': '2018-04-05 17:49:30.196894 2866', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '95159'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '95264'), ('function', 'dequeue_task'), ('message_id', '95264'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '95264'), ('message_id', '2018-04-05 17:49:48.389443 3659'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196963 2870', 'function': 'tasks.mapper', 'args': ('mournful',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196963 2870', 'task_id': '2018-04-05 17:49:30.196963 2870'}, 30)), ('sender', 'Client_366')])
    
    
    Sending 503 bytes
    Message:
    OrderedDict([('correlation_id', '95264'), ('message_id', '2018-04-05 17:49:48.389443 3659'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196963 2870', 'function': 'tasks.mapper', 'args': ('mournful',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196963 2870', 'task_id': '2018-04-05 17:49:30.196963 2870'}, 30)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '95341'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Lamb', 1], 'function': 'tasks.mapper', 'args': ['Lamb'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196913 2867', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196913 2867', 'correlation_id': '2018-04-05 17:49:30.196913 2867', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '95341'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '95447'), ('function', 'dequeue_task'), ('message_id', '95447'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '95447'), ('message_id', '2018-04-05 17:49:48.655267 3670'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196982 2871', 'function': 'tasks.mapper', 'args': ('tone',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196982 2871', 'task_id': '2018-04-05 17:49:30.196982 2871'}, 29)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '95447'), ('message_id', '2018-04-05 17:49:48.655267 3670'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196982 2871', 'function': 'tasks.mapper', 'args': ('tone',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196982 2871', 'task_id': '2018-04-05 17:49:30.196982 2871'}, 29)), ('sender', 'Client_366')])
    
    
    Data received: 534 bytes
    Message:
    OrderedDict([('correlation_id', '95731'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['in'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196929 2868', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196929 2868', 'correlation_id': '2018-04-05 17:49:30.196929 2868', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '95731'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '95836'), ('function', 'dequeue_task'), ('message_id', '95836'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '95836'), ('message_id', '2018-04-05 17:49:48.885858 3680'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196998 2872', 'function': 'tasks.mapper', 'args': ('of',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196998 2872', 'task_id': '2018-04-05 17:49:30.196998 2872'}, 28)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '95836'), ('message_id', '2018-04-05 17:49:48.885858 3680'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.196998 2872', 'function': 'tasks.mapper', 'args': ('of',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.196998 2872', 'task_id': '2018-04-05 17:49:30.196998 2872'}, 28)), ('sender', 'Client_366')])
    
    
    Data received: 533 bytes
    Message:
    OrderedDict([('correlation_id', '95952'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['a'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196947 2869', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196947 2869', 'correlation_id': '2018-04-05 17:49:30.196947 2869', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '95952'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '96058'), ('function', 'dequeue_task'), ('message_id', '96058'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '96058'), ('message_id', '2018-04-05 17:49:49.246413 3696'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197014 2873', 'function': 'tasks.mapper', 'args': ('voice',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197014 2873', 'task_id': '2018-04-05 17:49:30.197014 2873'}, 27)), ('sender', 'Client_366')])
    
    
    Sending 500 bytes
    Message:
    OrderedDict([('correlation_id', '96058'), ('message_id', '2018-04-05 17:49:49.246413 3696'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197014 2873', 'function': 'tasks.mapper', 'args': ('voice',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197014 2873', 'task_id': '2018-04-05 17:49:30.197014 2873'}, 27)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '96499'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['tone', 1], 'function': 'tasks.mapper', 'args': ['tone'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196982 2871', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196982 2871', 'correlation_id': '2018-04-05 17:49:30.196982 2871', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '96499'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '96608'), ('function', 'dequeue_task'), ('message_id', '96608'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '96608'), ('message_id', '2018-04-05 17:49:49.530838 3708'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197031 2874', 'function': 'tasks.mapper', 'args': ('"I',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197031 2874', 'task_id': '2018-04-05 17:49:30.197031 2874'}, 26)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '96608'), ('message_id', '2018-04-05 17:49:49.530838 3708'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197031 2874', 'function': 'tasks.mapper', 'args': ('"I',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197031 2874', 'task_id': '2018-04-05 17:49:30.197031 2874'}, 26)), ('sender', 'Client_366')])
    
    
    Data received: 534 bytes
    Message:
    OrderedDict([('correlation_id', '96669'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['of'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196998 2872', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196998 2872', 'correlation_id': '2018-04-05 17:49:30.196998 2872', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '96669'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 551 bytes
    Message:
    OrderedDict([('correlation_id', '96641'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['mournful', 1], 'function': 'tasks.mapper', 'args': ['mournful'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.196963 2870', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.196963 2870', 'correlation_id': '2018-04-05 17:49:30.196963 2870', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '96641'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '96774'), ('function', 'dequeue_task'), ('message_id', '96774'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '96774'), ('message_id', '2018-04-05 17:49:49.862043 3723'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197048 2875', 'function': 'tasks.mapper', 'args': ('was',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197048 2875', 'task_id': '2018-04-05 17:49:30.197048 2875'}, 25)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '96774'), ('message_id', '2018-04-05 17:49:49.862043 3723'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197048 2875', 'function': 'tasks.mapper', 'args': ('was',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197048 2875', 'task_id': '2018-04-05 17:49:30.197048 2875'}, 25)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '96749'), ('function', 'dequeue_task'), ('message_id', '96749'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '96749'), ('message_id', '2018-04-05 17:49:49.992195 3728'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197063 2876', 'function': 'tasks.mapper', 'args': ('not',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197063 2876', 'task_id': '2018-04-05 17:49:30.197063 2876'}, 24)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '96749'), ('message_id', '2018-04-05 17:49:49.992195 3728'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197063 2876', 'function': 'tasks.mapper', 'args': ('not',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197063 2876', 'task_id': '2018-04-05 17:49:30.197063 2876'}, 24)), ('sender', 'Client_366')])
    
    
    Data received: 545 bytes
    Message:
    OrderedDict([('correlation_id', '96997'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['voice', 1], 'function': 'tasks.mapper', 'args': ['voice'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197014 2873', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197014 2873', 'correlation_id': '2018-04-05 17:49:30.197014 2873', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '96997'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '97105'), ('function', 'dequeue_task'), ('message_id', '97105'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '97105'), ('message_id', '2018-04-05 17:49:50.269314 3740'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197081 2877', 'function': 'tasks.mapper', 'args': ('then',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197081 2877', 'task_id': '2018-04-05 17:49:30.197081 2877'}, 23)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '97105'), ('message_id', '2018-04-05 17:49:50.269314 3740'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197081 2877', 'function': 'tasks.mapper', 'args': ('then',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197081 2877', 'task_id': '2018-04-05 17:49:30.197081 2877'}, 23)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '97297'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['"I'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197031 2874', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197031 2874', 'correlation_id': '2018-04-05 17:49:30.197031 2874', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '97297'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '97402'), ('function', 'dequeue_task'), ('message_id', '97402'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '97402'), ('message_id', '2018-04-05 17:49:50.613631 3754'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197097 2878', 'function': 'tasks.mapper', 'args': ('born"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197097 2878', 'task_id': '2018-04-05 17:49:30.197097 2878'}, 22)), ('sender', 'Client_366')])
    
    
    Sending 501 bytes
    Message:
    OrderedDict([('correlation_id', '97402'), ('message_id', '2018-04-05 17:49:50.613631 3754'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197097 2878', 'function': 'tasks.mapper', 'args': ('born"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197097 2878', 'task_id': '2018-04-05 17:49:30.197097 2878'}, 22)), ('sender', 'Client_366')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '97748'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['was'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197048 2875', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197048 2875', 'correlation_id': '2018-04-05 17:49:30.197048 2875', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '97748'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '97723'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['not'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197063 2876', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197063 2876', 'correlation_id': '2018-04-05 17:49:30.197063 2876', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '97723'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '97854'), ('function', 'dequeue_task'), ('message_id', '97854'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '97854'), ('message_id', '2018-04-05 17:49:50.983469 3770'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197113 2879', 'function': 'tasks.mapper', 'args': ('Then',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197113 2879', 'task_id': '2018-04-05 17:49:30.197113 2879'}, 21)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '97854'), ('message_id', '2018-04-05 17:49:50.983469 3770'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197113 2879', 'function': 'tasks.mapper', 'args': ('Then',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197113 2879', 'task_id': '2018-04-05 17:49:30.197113 2879'}, 21)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '97829'), ('function', 'dequeue_task'), ('message_id', '97829'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '97829'), ('message_id', '2018-04-05 17:49:51.160583 3777'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197130 2880', 'function': 'tasks.mapper', 'args': ('said',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197130 2880', 'task_id': '2018-04-05 17:49:30.197130 2880'}, 20)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '97829'), ('message_id', '2018-04-05 17:49:51.160583 3777'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197130 2880', 'function': 'tasks.mapper', 'args': ('said',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197130 2880', 'task_id': '2018-04-05 17:49:30.197130 2880'}, 20)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '98235'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['then', 1], 'function': 'tasks.mapper', 'args': ['then'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197081 2877', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197081 2877', 'correlation_id': '2018-04-05 17:49:30.197081 2877', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '98235'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '98343'), ('function', 'dequeue_task'), ('message_id', '98343'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '98343'), ('message_id', '2018-04-05 17:49:51.403276 3787'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197148 2881', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197148 2881', 'task_id': '2018-04-05 17:49:30.197148 2881'}, 19)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '98343'), ('message_id', '2018-04-05 17:49:51.403276 3787'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197148 2881', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197148 2881', 'task_id': '2018-04-05 17:49:30.197148 2881'}, 19)), ('sender', 'Client_366')])
    
    
    Data received: 547 bytes
    Message:
    OrderedDict([('correlation_id', '98376'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['born"', 1], 'function': 'tasks.mapper', 'args': ['born"'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197097 2878', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197097 2878', 'correlation_id': '2018-04-05 17:49:30.197097 2878', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '98376'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '98484'), ('function', 'dequeue_task'), ('message_id', '98484'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '98484'), ('message_id', '2018-04-05 17:49:51.756229 3803'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197165 2882', 'function': 'tasks.mapper', 'args': ('Wolf',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197165 2882', 'task_id': '2018-04-05 17:49:30.197165 2882'}, 18)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '98484'), ('message_id', '2018-04-05 17:49:51.756229 3803'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197165 2882', 'function': 'tasks.mapper', 'args': ('Wolf',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197165 2882', 'task_id': '2018-04-05 17:49:30.197165 2882'}, 18)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '99284'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['said', 1], 'function': 'tasks.mapper', 'args': ['said'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197130 2880', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197130 2880', 'correlation_id': '2018-04-05 17:49:30.197130 2880', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '99284'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '99389'), ('function', 'dequeue_task'), ('message_id', '99389'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '99389'), ('message_id', '2018-04-05 17:49:52.221607 3824'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197181 2883', 'function': 'tasks.mapper', 'args': ('"You',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197181 2883', 'task_id': '2018-04-05 17:49:30.197181 2883'}, 17)), ('sender', 'Client_366')])
    
    
    Sending 500 bytes
    Message:
    OrderedDict([('correlation_id', '99389'), ('message_id', '2018-04-05 17:49:52.221607 3824'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197181 2883', 'function': 'tasks.mapper', 'args': ('"You',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197181 2883', 'task_id': '2018-04-05 17:49:30.197181 2883'}, 17)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '99512'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Then', 1], 'function': 'tasks.mapper', 'args': ['Then'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197113 2879', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197113 2879', 'correlation_id': '2018-04-05 17:49:30.197113 2879', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '99512'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '99621'), ('function', 'dequeue_task'), ('message_id', '99621'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '99621'), ('message_id', '2018-04-05 17:49:52.481600 3835'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197197 2884', 'function': 'tasks.mapper', 'args': ('feed',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197197 2884', 'task_id': '2018-04-05 17:49:30.197197 2884'}, 16)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '99621'), ('message_id', '2018-04-05 17:49:52.481600 3835'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197197 2884', 'function': 'tasks.mapper', 'args': ('feed',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197197 2884', 'task_id': '2018-04-05 17:49:30.197197 2884'}, 16)), ('sender', 'Client_366')])
    
    
    Data received: 543 bytes
    Message:
    OrderedDict([('correlation_id', '99496'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Wolf', 1], 'function': 'tasks.mapper', 'args': ['Wolf'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197165 2882', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197165 2882', 'correlation_id': '2018-04-05 17:49:30.197165 2882', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '99496'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 535 bytes
    Message:
    OrderedDict([('correlation_id', '99152'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['the'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197148 2881', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197148 2881', 'correlation_id': '2018-04-05 17:49:30.197148 2881', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '99152'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '99258'), ('function', 'dequeue_task'), ('message_id', '99258'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '99258'), ('message_id', '2018-04-05 17:49:52.885371 3852'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197214 2885', 'function': 'tasks.mapper', 'args': ('in',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197214 2885', 'task_id': '2018-04-05 17:49:30.197214 2885'}, 15)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '99258'), ('message_id', '2018-04-05 17:49:52.885371 3852'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197214 2885', 'function': 'tasks.mapper', 'args': ('in',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197214 2885', 'task_id': '2018-04-05 17:49:30.197214 2885'}, 15)), ('sender', 'Client_366')])
    
    
    Data received: 223 bytes
    Message:
    OrderedDict([('correlation_id', '99603'), ('function', 'dequeue_task'), ('message_id', '99603'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '99603'), ('message_id', '2018-04-05 17:49:53.002073 3857'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197230 2886', 'function': 'tasks.mapper', 'args': ('my',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197230 2886', 'task_id': '2018-04-05 17:49:30.197230 2886'}, 14)), ('sender', 'Client_366')])
    
    
    Sending 497 bytes
    Message:
    OrderedDict([('correlation_id', '99603'), ('message_id', '2018-04-05 17:49:53.002073 3857'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197230 2886', 'function': 'tasks.mapper', 'args': ('my',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197230 2886', 'task_id': '2018-04-05 17:49:30.197230 2886'}, 14)), ('sender', 'Client_366')])
    
    
    Data received: 547 bytes
    Message:
    OrderedDict([('correlation_id', '100119'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['"You', 1], 'function': 'tasks.mapper', 'args': ['"You'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197181 2883', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197181 2883', 'correlation_id': '2018-04-05 17:49:30.197181 2883', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '100119'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '100226'), ('function', 'dequeue_task'), ('message_id', '100226'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '100226'), ('message_id', '2018-04-05 17:49:53.279003 3869'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197245 2887', 'function': 'tasks.mapper', 'args': ('pasture"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197245 2887', 'task_id': '2018-04-05 17:49:30.197245 2887'}, 13)), ('sender', 'Client_366')])
    
    
    Sending 505 bytes
    Message:
    OrderedDict([('correlation_id', '100226'), ('message_id', '2018-04-05 17:49:53.279003 3869'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197245 2887', 'function': 'tasks.mapper', 'args': ('pasture"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197245 2887', 'task_id': '2018-04-05 17:49:30.197245 2887'}, 13)), ('sender', 'Client_366')])
    
    
    Data received: 545 bytes
    Message:
    OrderedDict([('correlation_id', '100398'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['feed', 1], 'function': 'tasks.mapper', 'args': ['feed'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197197 2884', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197197 2884', 'correlation_id': '2018-04-05 17:49:30.197197 2884', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '100398'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '100505'), ('function', 'dequeue_task'), ('message_id', '100505'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '100505'), ('message_id', '2018-04-05 17:49:53.511272 3879'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197261 2888', 'function': 'tasks.mapper', 'args': ('"No',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197261 2888', 'task_id': '2018-04-05 17:49:30.197261 2888'}, 12)), ('sender', 'Client_366')])
    
    
    Sending 500 bytes
    Message:
    OrderedDict([('correlation_id', '100505'), ('message_id', '2018-04-05 17:49:53.511272 3879'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197261 2888', 'function': 'tasks.mapper', 'args': ('"No',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197261 2888', 'task_id': '2018-04-05 17:49:30.197261 2888'}, 12)), ('sender', 'Client_366')])
    
    
    Data received: 555 bytes
    Message:
    OrderedDict([('correlation_id', '101132'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['pasture"', 1], 'function': 'tasks.mapper', 'args': ['pasture"'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197245 2887', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197245 2887', 'correlation_id': '2018-04-05 17:49:30.197245 2887', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '101132'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '101241'), ('function', 'dequeue_task'), ('message_id', '101241'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '101241'), ('message_id', '2018-04-05 17:49:54.240128 3914'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197276 2889', 'function': 'tasks.mapper', 'args': ('good',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197276 2889', 'task_id': '2018-04-05 17:49:30.197276 2889'}, 11)), ('sender', 'Client_366')])
    
    
    Sending 500 bytes
    Message:
    OrderedDict([('correlation_id', '101241'), ('message_id', '2018-04-05 17:49:54.240128 3914'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197276 2889', 'function': 'tasks.mapper', 'args': ('good',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197276 2889', 'task_id': '2018-04-05 17:49:30.197276 2889'}, 11)), ('sender', 'Client_366')])
    
    
    Data received: 538 bytes
    Message:
    OrderedDict([('correlation_id', '101329'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['"No'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197261 2888', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197261 2888', 'correlation_id': '2018-04-05 17:49:30.197261 2888', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '101329'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '101435'), ('function', 'dequeue_task'), ('message_id', '101435'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '101435'), ('message_id', '2018-04-05 17:49:54.542205 3927'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197295 2890', 'function': 'tasks.mapper', 'args': ('sir"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197295 2890', 'task_id': '2018-04-05 17:49:30.197295 2890'}, 10)), ('sender', 'Client_366')])
    
    
    Sending 501 bytes
    Message:
    OrderedDict([('correlation_id', '101435'), ('message_id', '2018-04-05 17:49:54.542205 3927'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197295 2890', 'function': 'tasks.mapper', 'args': ('sir"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197295 2890', 'task_id': '2018-04-05 17:49:30.197295 2890'}, 10)), ('sender', 'Client_366')])
    
    
    Data received: 536 bytes
    Message:
    OrderedDict([('correlation_id', '100661'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['my'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197230 2886', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197230 2886', 'correlation_id': '2018-04-05 17:49:30.197230 2886', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '100661'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 536 bytes
    Message:
    OrderedDict([('correlation_id', '100718'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['in'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197214 2885', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197214 2885', 'correlation_id': '2018-04-05 17:49:30.197214 2885', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '100718'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '100767'), ('function', 'dequeue_task'), ('message_id', '100767'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '100767'), ('message_id', '2018-04-05 17:49:54.967908 3946'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197326 2891', 'function': 'tasks.mapper', 'args': ('grossly',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197326 2891', 'task_id': '2018-04-05 17:49:30.197326 2891'}, 9)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '100767'), ('message_id', '2018-04-05 17:49:54.967908 3946'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197326 2891', 'function': 'tasks.mapper', 'args': ('grossly',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197326 2891', 'task_id': '2018-04-05 17:49:30.197326 2891'}, 9)), ('sender', 'Client_366')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '100823'), ('function', 'dequeue_task'), ('message_id', '100823'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '100823'), ('message_id', '2018-04-05 17:49:55.115084 3953'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197344 2892', 'function': 'tasks.mapper', 'args': ('replied',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197344 2892', 'task_id': '2018-04-05 17:49:30.197344 2892'}, 8)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '100823'), ('message_id', '2018-04-05 17:49:55.115084 3953'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197344 2892', 'function': 'tasks.mapper', 'args': ('replied',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197344 2892', 'task_id': '2018-04-05 17:49:30.197344 2892'}, 8)), ('sender', 'Client_366')])
    
    
    Data received: 545 bytes
    Message:
    OrderedDict([('correlation_id', '102119'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['good', 1], 'function': 'tasks.mapper', 'args': ['good'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197276 2889', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197276 2889', 'correlation_id': '2018-04-05 17:49:30.197276 2889', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '102119'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '102226'), ('function', 'dequeue_task'), ('message_id', '102226'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '102226'), ('message_id', '2018-04-05 17:49:55.392582 3964'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197368 2893', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197368 2893', 'task_id': '2018-04-05 17:49:30.197368 2893'}, 7)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '102226'), ('message_id', '2018-04-05 17:49:55.392582 3964'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197368 2893', 'function': 'tasks.mapper', 'args': ('the',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197368 2893', 'task_id': '2018-04-05 17:49:30.197368 2893'}, 7)), ('sender', 'Client_366')])
    
    
    Data received: 551 bytes
    Message:
    OrderedDict([('correlation_id', '102780'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['grossly', 1], 'function': 'tasks.mapper', 'args': ['grossly'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197326 2891', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197326 2891', 'correlation_id': '2018-04-05 17:49:30.197326 2891', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '102780'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '102889'), ('function', 'dequeue_task'), ('message_id', '102889'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '102889'), ('message_id', '2018-04-05 17:49:55.740529 3979'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197386 2894', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197386 2894', 'task_id': '2018-04-05 17:49:30.197386 2894'}, 6)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '102889'), ('message_id', '2018-04-05 17:49:55.740529 3979'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197386 2894', 'function': 'tasks.mapper', 'args': ('Lamb',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197386 2894', 'task_id': '2018-04-05 17:49:30.197386 2894'}, 6)), ('sender', 'Client_366')])
    
    
    Data received: 551 bytes
    Message:
    OrderedDict([('correlation_id', '102857'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['replied', 1], 'function': 'tasks.mapper', 'args': ['replied'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197344 2892', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197344 2892', 'correlation_id': '2018-04-05 17:49:30.197344 2892', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '102857'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '102966'), ('function', 'dequeue_task'), ('message_id', '102966'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '102966'), ('message_id', '2018-04-05 17:49:55.974717 3989'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197404 2895', 'function': 'tasks.mapper', 'args': ('"I',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197404 2895', 'task_id': '2018-04-05 17:49:30.197404 2895'}, 5)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '102966'), ('message_id', '2018-04-05 17:49:55.974717 3989'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197404 2895', 'function': 'tasks.mapper', 'args': ('"I',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197404 2895', 'task_id': '2018-04-05 17:49:30.197404 2895'}, 5)), ('sender', 'Client_366')])
    
    
    Data received: 547 bytes
    Message:
    OrderedDict([('correlation_id', '102991'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['sir"', 1], 'function': 'tasks.mapper', 'args': ['sir"'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197295 2890', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197295 2890', 'correlation_id': '2018-04-05 17:49:30.197295 2890', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '102991'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '103098'), ('function', 'dequeue_task'), ('message_id', '103098'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '103098'), ('message_id', '2018-04-05 17:49:56.228615 3999'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197421 2896', 'function': 'tasks.mapper', 'args': ('have',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197421 2896', 'task_id': '2018-04-05 17:49:30.197421 2896'}, 4)), ('sender', 'Client_366')])
    
    
    Sending 499 bytes
    Message:
    OrderedDict([('correlation_id', '103098'), ('message_id', '2018-04-05 17:49:56.228615 3999'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197421 2896', 'function': 'tasks.mapper', 'args': ('have',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197421 2896', 'task_id': '2018-04-05 17:49:30.197421 2896'}, 4)), ('sender', 'Client_366')])
    
    
    Data received: 537 bytes
    Message:
    OrderedDict([('correlation_id', '103161'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['the'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197368 2893', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197368 2893', 'correlation_id': '2018-04-05 17:49:30.197368 2893', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '103161'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '103267'), ('function', 'dequeue_task'), ('message_id', '103267'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '103267'), ('message_id', '2018-04-05 17:49:56.502495 4011'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197438 2897', 'function': 'tasks.mapper', 'args': ('not',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197438 2897', 'task_id': '2018-04-05 17:49:30.197438 2897'}, 3)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '103267'), ('message_id', '2018-04-05 17:49:56.502495 4011'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197438 2897', 'function': 'tasks.mapper', 'args': ('not',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197438 2897', 'task_id': '2018-04-05 17:49:30.197438 2897'}, 3)), ('sender', 'Client_366')])
    
    
    Data received: 545 bytes
    Message:
    OrderedDict([('correlation_id', '103395'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['Lamb', 1], 'function': 'tasks.mapper', 'args': ['Lamb'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197386 2894', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197386 2894', 'correlation_id': '2018-04-05 17:49:30.197386 2894', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '103395'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '103503'), ('function', 'dequeue_task'), ('message_id', '103503'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '103503'), ('message_id', '2018-04-05 17:49:56.772771 4022'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197453 2898', 'function': 'tasks.mapper', 'args': ('yet',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197453 2898', 'task_id': '2018-04-05 17:49:30.197453 2898'}, 2)), ('sender', 'Client_366')])
    
    
    Sending 498 bytes
    Message:
    OrderedDict([('correlation_id', '103503'), ('message_id', '2018-04-05 17:49:56.772771 4022'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197453 2898', 'function': 'tasks.mapper', 'args': ('yet',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197453 2898', 'task_id': '2018-04-05 17:49:30.197453 2898'}, 2)), ('sender', 'Client_366')])
    
    
    Data received: 537 bytes
    Message:
    OrderedDict([('correlation_id', '103795'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['"I'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197404 2895', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197404 2895', 'correlation_id': '2018-04-05 17:49:30.197404 2895', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '103795'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '103901'), ('function', 'dequeue_task'), ('message_id', '103901'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '103901'), ('message_id', '2018-04-05 17:49:57.029533 4033'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197473 2899', 'function': 'tasks.mapper', 'args': ('tasted',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197473 2899', 'task_id': '2018-04-05 17:49:30.197473 2899'}, 1)), ('sender', 'Client_366')])
    
    
    Sending 501 bytes
    Message:
    OrderedDict([('correlation_id', '103901'), ('message_id', '2018-04-05 17:49:57.029533 4033'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197473 2899', 'function': 'tasks.mapper', 'args': ('tasted',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197473 2899', 'task_id': '2018-04-05 17:49:30.197473 2899'}, 1)), ('sender', 'Client_366')])
    
    
    Data received: 545 bytes
    Message:
    OrderedDict([('correlation_id', '104077'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['have', 1], 'function': 'tasks.mapper', 'args': ['have'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197421 2896', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197421 2896', 'correlation_id': '2018-04-05 17:49:30.197421 2896', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '104077'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '104185'), ('function', 'dequeue_task'), ('message_id', '104185'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '104185'), ('message_id', '2018-04-05 17:49:57.272097 4043'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197492 2900', 'function': 'tasks.mapper', 'args': ('grass"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197492 2900', 'task_id': '2018-04-05 17:49:30.197492 2900'}, 0)), ('sender', 'Client_366')])
    
    
    Sending 502 bytes
    Message:
    OrderedDict([('correlation_id', '104185'), ('message_id', '2018-04-05 17:49:57.272097 4043'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890c49'), ('reply_to', 'Client_366'), ('result', ({'sender': 'Client_366', 'message_type': 'function', 'message_id': '2018-04-05 17:49:30.197492 2900', 'function': 'tasks.mapper', 'args': ('grass"',), 'need_result': True, 'reply_to': 'Client_366', 'correlation_id': '2018-04-05 17:49:30.197492 2900', 'task_id': '2018-04-05 17:49:30.197492 2900'}, 0)), ('sender', 'Client_366')])
    
    
    Data received: 537 bytes
    Message:
    OrderedDict([('correlation_id', '104361'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['not'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197438 2897', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197438 2897', 'correlation_id': '2018-04-05 17:49:30.197438 2897', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '104361'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '104467'), ('function', 'dequeue_task'), ('message_id', '104467'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '104467'), ('message_id', '2018-04-05 17:49:57.554967 4054'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 210 bytes
    Message:
    OrderedDict([('correlation_id', '104467'), ('message_id', '2018-04-05 17:49:57.554967 4054'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 537 bytes
    Message:
    OrderedDict([('correlation_id', '104538'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': None, 'function': 'tasks.mapper', 'args': ['yet'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197453 2898', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197453 2898', 'correlation_id': '2018-04-05 17:49:30.197453 2898', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '104538'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '104643'), ('function', 'dequeue_task'), ('message_id', '104643'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890499'), ('sender', 'NodeMCU_b4e62d890499')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '104643'), ('message_id', '2018-04-05 17:49:57.770844 4064'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 210 bytes
    Message:
    OrderedDict([('correlation_id', '104643'), ('message_id', '2018-04-05 17:49:57.770844 4064'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890499'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 549 bytes
    Message:
    OrderedDict([('correlation_id', '104873'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['tasted', 1], 'function': 'tasks.mapper', 'args': ['tasted'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197473 2899', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197473 2899', 'correlation_id': '2018-04-05 17:49:30.197473 2899', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '104873'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '104981'), ('function', 'dequeue_task'), ('message_id', '104981'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890a95'), ('sender', 'NodeMCU_b4e62d890a95')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '104981'), ('message_id', '2018-04-05 17:49:57.994100 4073'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 210 bytes
    Message:
    OrderedDict([('correlation_id', '104981'), ('message_id', '2018-04-05 17:49:57.994100 4073'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d890a95'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 551 bytes
    Message:
    OrderedDict([('correlation_id', '105699'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ["Aesop's", 1], 'function': 'tasks.mapper', 'args': ["Aesop's"], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.141388 2805', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.141388 2805', 'correlation_id': '2018-04-05 17:49:30.141388 2805', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '105699'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Data received: 225 bytes
    Message:
    OrderedDict([('correlation_id', '105807'), ('function', 'dequeue_task'), ('message_id', '105807'), ('message_type', 'function'), ('need_result', True), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d891371'), ('sender', 'NodeMCU_b4e62d891371')])
    
    
    Processed result:
    OrderedDict([('correlation_id', '105807'), ('message_id', '2018-04-05 17:49:59.144429 4133'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Sending 210 bytes
    Message:
    OrderedDict([('correlation_id', '105807'), ('message_id', '2018-04-05 17:49:59.144429 4133'), ('message_type', 'result'), ('receiver', 'NodeMCU_b4e62d891371'), ('reply_to', 'Client_366'), ('result', (None, 0)), ('sender', 'Client_366')])
    
    
    Data received: 551 bytes
    Message:
    OrderedDict([('correlation_id', '105191'), ('function', 'enqueue_result'), ('kwargs', {'message': {'result': ['grass"', 1], 'function': 'tasks.mapper', 'args': ['grass"'], 'reply_to': 'Client_366', 'message_id': '2018-04-05 17:49:30.197492 2900', 'message_type': 'result', 'task_id': '2018-04-05 17:49:30.197492 2900', 'correlation_id': '2018-04-05 17:49:30.197492 2900', 'need_result': True, 'sender': 'Client_366'}}), ('message_id', '105191'), ('message_type', 'function'), ('receiver', 'Client_366'), ('reply_to', 'NodeMCU_b4e62d890c49'), ('sender', 'NodeMCU_b4e62d890c49')])
    
    ********** result:
    words count: 94
    
    [('Lamb', 5), ('grossly', 3), ('Wolf', 2), ('year', 1), ('with', 1), ('voice', 1), ('violent', 1), ('tone', 1), ('thus', 1), ('then', 1)]
    **********
    


```python
# Stopping
the_client.stop()
the_client = None
print('\n[________________ Demo stopped ________________]\n')
```

    [Closed: ('123.240.210.68', 1883)]
    
    [________________ Demo stopped ________________]
    
    