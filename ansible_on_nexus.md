# 拓扑图
![01](/images/ansible-on-nexus-1.PNG)
# 需求
利用ansible管理网络设备,并对设备进行配置
1. 在NXOS交换机上开启nxapi的特性 feature nxapi
使用命令show feature | in enable 查看特性是否已经开启
2. 登陆ansible的服务器,查看ansible的认证信息cat .netauth,并查看hosts文件信息，可以看到4台NXOS设备都被记录在资产表中了。
![02](./inventory.PNG)
3. 该实验环境下，ansible已经安装完成，可以使用ansible --version 查看安装的版本信息
![03](/images/ansible-version.PNG)
## 收集设备上的信息
在ansible中我们可以制定一系列的操作，可以称为playbook，然后运行这个playbook，就可以按预先设定好的顺序执行操作。
下图为我们要运行的playbook详细信息。在这个playbook中，分别做了查看cdp信息，查看管理路由，将信息记录在本地 总共三个操作。
在ansible中有特定的模块对应特定系统，比如针对NX-OS，ansible就有一个专门的NX-OS模块，存储了对系统能进行的相关操作，不用再写出详细的命令。
![04](/images/gather-data-yml.PNG)
对于运行playbook需要使用ansible-playbook命令来调用，如果配置有改动，那么会显示黄色，如果没有任何改动，那么会显示绿色。
__ansible有等幂性，即使同一条命令运行许多次，但是配置改动只会执行一次，不会执行多次。__
![05](/images/playbook-process.PNG)
当我们运行完playbook之后，可以看到在data文件夹下已经生成了4台设备上的相关配置信息。
当生成本地文件时，是用指定的templates/data.j2为模板来保存的。
![06](/images/templates.PNG)
## 对NX-OS进行配置
![07](/images/structure.PNG)
对于hosts文件来说，里面每一项都有一个对应的vars，单独的host的vars存放在host_vars文件夹下，group的属性存放在group_vars文件夹下。
当playbook运行的时候，调用hosts中的资产，会根据他们所属的group或者他们本身的vars，来读取相应的配置文件。
