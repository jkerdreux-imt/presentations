---
theme: default
paginate: true
marp: true
footer: "Jérôme Kerdreux"
---

# Programmation asynchrone avancée en Python


---
## Coroutines vs Threads et notre ami GIL  
- GIL : Global Interpreter Lock (GC)
=> Les threads en Python sont synchrones

---
## Gevent - Greenlet
- bibliothèque de coroutine basée sur [greenlet](https://greenlet.readthedocs.io/en/latest/) orientée réseau
- première version : 2009


```py
import gevent

class Hello(gevent.Greenlet):
    def __init__(self, name):
        gevent.Greenlet.__init__(self)
        self.name = name 

    def _run(self):
        while 1:
            print(f"Hello {self.name}")
            gevent.sleep(len(self.name))

Hello.spawn('Bob')
Hello.spawn('Alice')
gevent.sleep(10)
```

---
## Gevent - decorator
- s'intègre facilement dans du code existant, notamment avec un décorateur
```py
import gevent
from decorator import decorator

@decorator
def spawn(func,*args,**kwargs):
    gevent.spawn(func,*args,**kwargs)

@spawn
def hello(name):
    while 1:
        print(f"Hello {name}")
        gevent.sleep(len(name))

hello('Bob')
hello('Alice')
gevent.sleep(10)
```

---
## Gevent - monkey patching
- possibilité de "monkey patcher" les bibliothèques standards (socket, select ...)
- et donc également les modules externes écrit en Python.
```py
from gevent import monkey; monkey.patch_all(thread=False)
import gevent
...
@spawn
def test():
	time.sleep(10) # non-blocant ! 
```

---
## Asyncio - create_task
- succède à twisted, asyncore, async ...
- mots clés : **async / await**
- API ~ stable depuis Py3.6

```py
import asyncio

async def hello(name):
    while 1:
        print(f"Hello {name}")
        await asyncio.sleep(len(name))

async def main():
	# Tasks are used to schedule coroutines concurrently.
    t1 = asyncio.create_task(hello('Bob'))
    t2 = asyncio.create_task(hello('Alice'))
    await t1
    await t2

asyncio.run(main())
```


---
## Asyncio - blue or red
- une coroutine peut retrourner un résultat 
```py
fact = await factoriel(20)
```
- il existe quelques autres fonctions utiles : 
```py 
- loop = asyncio.new_event_loop()
- loop.create_task(foo())
- loop.run_until_complete(main())
- asyncio.all_tasks()
```

- WARNING : pas de await dans le constructeur (```__init__```)
- pas de await en dehors d'une fonction (REPL)
- blue / red functions => [What Color is Your Function? ](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)



___
## Asyncio - Event
```py
import asyncio

time_out = asyncio.Event()

async def hello(name):
    while not time_out.is_set():
        print(f"Hello {name}")
        await asyncio.sleep(len(name))

async def wait_time_out():
    await asyncio.sleep(10)
    time_out.set()

async def main():
    c1 = hello('Bob')
    c2 = hello('Alice')
    c3 = wait_time_out()
    await asyncio.gather(c1,c2,c3)

asyncio.run(main())
```


---
## Asyncio - Lock
```py

lock = asyncio.Lock()

async def write():
    while event.is_set():
        async with lock:
            values.append(random.randint(0,500))

async def read():
    global values
    async with lock:
        print(values)
        values = []
```


---
## Asyncio - IPython
```py
import asyncio
import IPython
from IPython.lib import backgroundjobs
import random

values = []
event = asyncio.Event()

async def test():
    while not event.is_set():
        values.append(random.randint(0,500))
        await asyncio.sleep(2)

def run():
    loop = asyncio.new_event_loop()
    t = loop.create_task(test())
    loop.run_until_complete(t)

jobs = backgroundjobs.BackgroundJobManager()
jobs.new(run)

IPython.embed(banner1="=Shell=", banner2="\n", colors="Linux", separate_in = '', autoawait = True)
event.set()
```


---
## Asyncio - modules
### aiohttp
```py
import asyncio
import aiohttp

async def test():
    session = aiohttp.ClientSession()
    try:
        response = await session.get('http://python.org')
        print(response.headers)
        await session.close()        
    except aiohttp.ClientConnectorError:
        print('Error')

asyncio.run(test())
```

---
## Asyncio - modules
### aioconsole 
```py
server = await aioconsole.start_interactive_server(host='localhost', port=6666)
```

```sh 
$ rlwrap nc localhost 6666
```

### aiotimeout 
```py
from aiotimeout import timeout

# Will raise an asyncio.TimeoutError
with timeout(1):
    await asyncio.sleep(1.5)
```

