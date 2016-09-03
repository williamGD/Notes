FLAASH 大气校正 error

![Alt Text](https://github.com/williamGD/Notes/blob/master/Pictures/FLAASHerror1.png)

原因分析：软件对输出/输入文件读写权限受限，如这里默认输出文件夹"C:\Users\dsbin\AppData\Local\Temp"读写权限受限，导致读取不到相应的文件

解决方案：更改默认输出文件夹