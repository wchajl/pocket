# nmcli
> **用于控制网络管理器的命令行工具**
> + 用途
>   1. 查看附近wifi 
>   2. 链接wifi 
>   3. 查看各个网卡当前的状态
>   4. 查看所有保存的网络链接


+ details:
    nmcli is a command-line tool for controlling NetworkManager and reporting network status. It can be utilized as a replacement for nm-applet or other graphical
    clients.  nmcli is used to create, display, edit, delete, activate, and deactivate network connections, as well as control and display network device status.
    
       
+ Typical uses include:
    1. Scripts: Utilize NetworkManager via nmcli instead of managing network connections manually.  nmcli supports a terse output format which is better suited
    for script processing. Note that NetworkManager can also execute scripts, called "dispatcher scripts", in response to network events. See NetworkManager(8) for details about these dispatcher scripts.
    2. Servers, headless machines, and terminals: nmcli can be used to control NetworkManager without a GUI, including creating, editing, starting and stopping network connections and viewing network status.


## 通用命令
### 整体状态(general)
    `nmcli general {status | hostname | permissions | logging} [ARGUMENTS...]`
>      Use this command to show NetworkManager status and permissions. You can
       also get and change system hostname, as well as NetworkManager logging
       level and domains.
+ 此命令拥有四个子命令 status hostname permissions logging
1. status 
    Show overall status of NetworkManager. This is the default action, when
    no additional command is provided for nmcli general.

    ```
    houston@houston:~/project/github/pocket/linux$ nmcli general status -c
    STATE   CONNECTIVITY  WIFI-HW  WIFI    WWAN-HW  WWAN
    已连接  完全          已启用   已启用  已启用   已启用
    ```
2. hostname [hostname]
    + 用于查看或修改系统域名，如果没有参数参数，则返回当前系统域名，如果传入参数，则会将该参数传入网络管理器，并将其设置为新的系统域名
    > Get and change system hostname. 
        With no arguments, this prints currently configured hostname.
        When you pass a hostname, it will be handed over to NetworkManager to be set as a new system hostname.
    > Note that the term "system" hostname may also be referred to as "persistent" or "static" by other programs or tools. 
      The hostname is stored in /etc/hostname file in most distributions.
      For example, systemd-hostnamed service uses the term "static" hostname and it only reads the /etc/hostname file when it starts.
3. permissions

4. logging [level param1] [domains params...]


## 网络控制命令(Networking control commands)
### 整体网络(networking)
    `nmcli networking {on | off | connectivity} [ARGUMENTS...]`
> 
    Query NetworkManager networking status, enable and disable networking.
+ 有三个命令，分别用于控制网络的链接/断开 以及查看当前网络链接状况
1. on/off 
    启用或禁用网络管理器管理的网络。会将所有由网络管理器管理的网络接口/网卡进行开启和关闭
2. connectivity [check]
    获取网络链接状态，可以使用`check`选项来让网络管理器复检网络链接。
    默认显示最近一次的网络链接状态，而不会复检
    - 所有可能的状态
        1. none 当前主机未链接到任何网络
        2. portal 
            > the host is behind a captive portal and cannot reach the full
               Internet.
        3. limited 连接到网络，但无法访问互联网
        4. full 连接到网络，可以访问互联网

        5. unknown 未知的连接状态
###  广播传输控制命令(Radio transmission control commands)
    `nmcli radio {all | wifi | wwan} [ARGUMENTS...]`
> Show radio switches status, or enable and disable the switches
1. wifi [on | off] 
    可以用来查看wifi状态或打开/关闭wifi
2. wwan [on | off]
    可以用来查看wwan状态或打开/关闭wwan
3. all [on | off]
    可以用来查看所有wifi和wwan的状态或将他们同时打开/关闭

### 活动监控命令 (Activity monitor)
    `nmcli monitor`
+ 用于观察网络活动。监控连接状态的变化，以及网卡、连接文件等等

### 连接管理命令 (connection)
`nmcli connection {show | up | down | modify | add | edit | clone | delete | monitor | reload | load | import | export} [ARGUMENTS...] `
+ 网络管理器会将所有的网络配置信息存储为连接`connections`,连接是用于描述如何建立网络链接的数据.
+ 只有当网络设备使用一个连接的配置信息创建/连接到网络时，一个连接的状态才是活跃的。
+ 一个设备可以使用多个连接，但在任意时刻只能有一个连接是活跃的。其他连接被用于在不同的网络和配置信息之间快速切换


### 设备管理命令(Device management Command)
    `nmcli device {status | show | set | connect | reapply | modify | disconnect | delete | monitor | wifi | lldp} [ARGUMENTS...]`
+ 显示和管理网络接口/网卡/网络设备
1. status
    打印当前网络设备的状态
2. show [ifname]
    用于显示网络设备的详细信息，如果不传入接口名称，则默认打印全部的网络接口
3. set [ifname] ifname [autoconnect {yes | no}] [managed {yes | no}]
    设置网络设备属性
4. connect ifname
    连接指定设备。网络管理器会自动搜索一个可用的连接并激活（会搜索没有设置auto connect的）
5. reapply ifname

6. modify ifname {option value | [+|-]setting.property value}...

7. disconnect ifname...

8. delete ifname...

9. monitor [ifname...]

10. wifi [list [--rescan | auto | no | yes]] [ifname ifname] [bssid BSSID]
    + 显示所有可用的wifi接入点。
    + ifname 和 bssid 选项可用于显示特定网络接口下指定bssid wifi接入点的信息
    + 默认情况下，nmcli 会确保显示的是30s以内的wifi接入点信息，并根据需要出发网络扫描
    + --rescan 参数可用于强制扫描接入点链接信息
    
11. wifi connect (B)SSID [password password] [wep-key-type {key | phrase}] [ifname ifname] [bssid BSSID] [name name] [private {yes | no}] [hidden {yes | no}]
    + 连接到SSID 或 BSSID 指定的wifi网络中。
    + 此命令会创建一个新的连接，并在一个设备上激活他。相当于在GUI界面上点击了一个wifi连接。
    + 此命令总是会创建一个新的连接，因此适合用于连接的新的wifi网络。如果链接已经存在，使用`nmcli connection up id name` 来启用该连接。
    + 注意目前只支持open,WEP 和WPA-PSK 网络
    + 假定 ip配置信息 可通过DHCP获取






