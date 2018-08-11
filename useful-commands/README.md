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
