# Useful commands
# Ubuntu
1. 在一个工程下面, 跨文件寻找一个短语或句子出现的文件跟对应的行数, 需要熟悉`find`与`grep`命令. _例如在`ORB-SLAM2`工程下所有`.cc`后缀文件中寻找`ORB-SLAM2:`字符串, 使用下面的命令:_
```bash
find <folder path> -type f -name "*.cc" | xargs grep "ORB-SLAM2:"
```
2. 想要批量重命名一个文件夹下的所有文件的名字,可以使用如下操作(示例中是将所有图片重名为按照顺序递增新的名字,统一后缀为`.jpg`):
```bash
cd <your floder dir>
i=1; for x in *; do mv $x $i.jpg; let i=i+1; done
```

3. 使用Python提供的`multiprocessing` 库有一个bug就是在使用多进程时候，主进程异常终止（e.g. 使用“Ctrl+C”强行退出），相应的数据加载进程可能无法正常退出。此时你发现主程序已经退出了，但是GPU显存和内存依旧被占用着，通过命令`top` 或者`ps aux` 依旧可以看到已经退出的程序，此时需要手动强行终止进程。建议使用的命令如下：

```bash
ps x | grep <cmdline> | awk '{print $1}' | xargs kill
```

* `ps x`：获取当前用户所有的进程；

* `grep <cmdline>`：找到已经停止的程序进程（e.g. 你是使用`python train.py`来启动的进程，那就写`grep 'pyhton train.py'`）；

* `awk '{print $i}'`： 获取进程的pid；

* `xargs kill`： 终止进程，根据需要可能要写成`xargs kill -9`强制终止。

  执行这个命令之前，建议打印确认进程

  ```bask
  ps x | grep <cmdline> | ps x
  ```


