---
layout: default
title: 目录变化监测实时同步
category: python
excerpt: Watching directory changes,upload to server instantly
tags: ["watchdog", "python"]
---

### {{ page.title }}
***

> **Win下有WinSCP，但是Mac没有比较好的实时同步工具，所以就写了此脚本。**
>
> 实时监测某个目录的文件变化，有更新即时上传到开发服务器，用于调试
>
> 跳过目录操作，不去删除服务器文件
>
> 用法: `python {script} {directory}`

{% highlight python linenos %}
#!/usr/bin/python
import time,sys,os
from watchdog.observers import Observer
from watchdog.events import PatternMatchingEventHandler,FileSystemEventHandler

RHOST="192.168.0.129"

class MyHandler(PatternMatchingEventHandler, FileSystemEventHandler):
    patterns = ["*"]

    def __init__(self, rhost):
        self.rhost = rhost
        self._ignore_directories = []
        self._ignore_patterns = []
        self._case_sensitive = True

    def process(self, event):
        #print event.src_path, event.event_type
        if event.event_type=='deleted' or event.is_directory:
            return
        lpath = event.src_path
        rpath = os.path.dirname(lpath)
        try:
            st = os.lstat(event.src_path)
            pip = os.popen("rsync -e 'ssh -i /path/to/.ssh/id_dsa -l root' -avz %s %s:%s" % (lpath,self.rhost,rpath))
            print("rsync -e 'ssh -i /path/to/.ssh/id_dsa -l root' -avz %s %s:%s/" % (lpath,self.rhost,rpath))
            print pip.read()
            pip.close()
        except Exception,e:
            print e
            pass

    def on_any_event(self, event):
        self.process(event)

if __name__=="__main__":
    args = sys.argv[1:]
    observer = Observer()
    observer.schedule(MyHandler(RHOST), path=args[0] if args else '.', recursive=True)
    observer.start()

    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()

    observer.join()
{% endhighlight %}
