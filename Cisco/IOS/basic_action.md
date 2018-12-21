# basic_action
## 实验一(通过ansible ad hoc命令查询设备的接口信息)
拓扑如下:
![01](./images/cisco/ios/topo.png)
首先在hosts文件内添加路由器的IP地址192.168.1.251
使用**ansible --list-host all**可以查看当前inventory的信息
![02](./images/cisco/ios/list-host.PNG)
使用**ansible 192.168.1.251 -m raw -a "sh ip int br" -u python -k**可以返回设备的接口信息
![03](./images/cisco/ios/basic_action_3.PNG)
* -m 用来指定模块.如果有被管理设备支持python的话,可以使用command和shell模块;但是如果老设备不支持的话,则需要使用raw模块来访问设备.
* -a 表示参数,表示在设备上要执行的命令,命令内容用单引号或者双引号括起来
* -u 表示使用的用户
* -k 提示用户输入密码
## 实验二(通过ansible playbook查询设备配置)
首先创建一个route的playbook,用来查询设备上的路由信息
![04](./images/cisco/ios/basic_action_4.png)
* name: playbook的名称
* hosts: 要执行的playbook的设备
* gather_facts: 默认自动执行,用来发现远程主机的各种已有参数.消耗时间较长,而且我们暂时用不到,可以设置为false不执行该操作.
* task: 表示我们在设备上要执行的操作
* - name：操作的名称
* raw: 模块名称
* register: 将执行命令后的输出结果保存,并存储在一个自定义的变量之中,这里为print_output
* - debug: var=print_output.stdout_lines将自定义变量中的内容打印出来
使用命令**ansible-playbook route.yml**执行命令(由于我在hosts文件中指定了设备的用户名和密码,所以在命令行中就不用指定了)
![05](./images/cisco/ios/basic_action_5.png)
## 实验三(使用playbook执行多条命令)
raw模块最大的缺点是只能执行一条命令，无法批量输入命令.但是现在ansible对于很多厂商的设备开发了相关的模块,可以更好的支持对设备的控制.比如针对cisco的IOS系统,有相对的模块.
![06](./images/cisco/ios/basic_action_6.png)
比如ios_command和ios_config模块,前者不支持configure模式下的命令,而后者支持.所以在选择模块的时候,需要有针对性.
首先创建一个multi_cmd.yml的playbook,用来查询接口信息和始终信息
![07](./images/cisco/ios/basic_action_7.png)
使用ansible-playbook执行命令之后,可以发现两条命令的结果都返回了.
## 实验四(在设备使用show run命令,并保存配置)
直接修改之前的playbook,并添加相应的操作即可.
![08](./images/cisco/ios/basic_action_8.png)
输出结果如下:在CLI下输出结果,并且可以在download文件夹下看到以设备IP命名的txt文件.
```
[root@localhost ansible]# ansible-playbook download.yml

PLAY [switch] ********************************************************************************************************************

TASK [show run] ******************************************************************************************************************
ok: [192.168.1.251]

TASK [debug] *********************************************************************************************************************
ok: [192.168.1.251] => {
    "output.stdout_lines": [
        [
            "Building configuration...",
            "",
            "Current configuration : 1953 bytes",
            "!",
            "! Last configuration change at 06:20:54 UTC Fri Dec 21 2018",
            ........
            "line vty 0 4",
            " login local",
            " transport input ssh",
            "!",
            "!",
            "end"
        ]
    ]
}

TASK [save and download] *********************************************************************************************************
changed: [192.168.1.251]

PLAY RECAP ***********************************************************************************************************************
192.168.1.251              : ok=3    changed=1    unreachable=0    failed=0  
```
当执行第二次执行的时候,会发现changed=0. 因为ansible的等幂性,有影响的操作只会执行一次.可以看到download文件夹下的文件创建时间始终是第一次运行playbook时创建的.
## 实验五(对设备进行配置,并保存)
```
#update_save.yml
---
- hosts: switch
  gather_facts: false
  connection: local
  tasks:
  - name: enable ospf
    ios_config:
      authorize: yes
      parents: router ospf 2
      lines:
        - network 192.168.1.0 0.0.0.0 area 0
      save_when: always
    register: output
  - debug: var=output
```
* 此处的authorize表示是否进入特权模式,默认是no,如果不设置为yes,运行playbook的时候就会报错.
* parents则表示为父配置,子配置则是在是父配置的前提下才可以配置.
* 保存配置是通过save_when: always来进行的. 总共有4个参数:always,modified,changed,never. 默认是never从不保存;如果是always，那么就是不管有没有修改配置都会保存;如果是modified,只要是对配置有修改,只有第一次才保存,后面都不会保存;如果是changed,只要是对配置进行修改,执行了多次,每次都会保存.
