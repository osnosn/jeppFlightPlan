Executable binary file.   

Write in rust on 2024-1-26. 用rust重写。   
Compiled by cargo.(rustup-1.26.0, rustc-1.70.0, cargo-1.70.0)   
Use rusqlite-0.30.0 crate.   
 | 【[Download 下载](https://github.com/osnosn/jeppFlightPlan/releases)】 v0.3.5 |   |
 |--------|-------|
 | jeppFlightPlan_aarch64-linux-gnu.gz      | arm 64bit linux 命令行的可执行文件 |
 | jeppFlightPlan_x86_64-linux-gnu.gz       | x86 64bit linux 命令行的可执行文件 |
 | jeppFlightPlan_x86_64-linux-musl.gz      | x86 64bit linux 命令行的可执行文件 |
 | jeppFlightPlan_mipsel-linux-musl-op22.gz | openwrt22, mpsel 命令行的可执行文件,动态链接 |
 | jeppFlightPlan_i686-gnu.exe.gz           | windows 32bit 命令行的可执行文件 |
 | jeppFlightPlan_x86_64-gnu.exe.gz         | windows 64bit 命令行的可执行文件 |
 
 ".gz" 是压缩文件，需要解压缩。   
 除mipsel是动态链接。其他的是静态链接文件。不依赖系统的动态库。    
 rust编译生成的windows程序，如果防毒误报。添加例外信任即可。   
 
 --------------
 执行程序，需要两个文件。  
 * ini 配置文件。默认文件名为 "jeppFlightPlan.ini"。可以用"-c"参数指定。   
   - 执行 `jeppFlightPlan  showini` 可以见到 ini 的示例。   
 * sqlite3 数据库文件。文件名写在ini文件中。   
   - 执行 `jeppFlightPlan  createdb` 可以得到db文件的样例。   
 
 带参数 --help 执行，可以看到详细的说明。   
   比如: amd64-linux-jeppFlightPlan --help    
```
Usage: jeppFlightPlan [-h | --help] [-c config.ini] [run | showini | createdb]
   Detail:
      -h        简略的命令行帮助
      --help     详细的帮助信息
      -c /path/conf.ini  指定ini配置文件
      run       运行服务
      showini   显示ini配置示例
      createdb  创建示例数据库 (db文件名在ini中配置)
 ----ini配置文件----
 #此配置文件必须是 UTF-8 格式.
 ;注释
 #服务设置
 [webserver]             #注释,"#"前必须有一个空格,才是注释
 #address=0.0.0.0:8080   #监听地址和端口
 #address=[::]:8080      # IPv6
 #address=[::1]:7878     # IPv6 localhost
 address=127.0.0.1:7878
 host_string=http://127.0.0.1:7878   #程序输出的内容中,链接到自身的地址.可以使用域名。末尾无"/".
 #没有使用反向代理时，"host_string=" 的域名/IP指向的是"address="的IP和端口。协议头为"http://"。
 #使用了反向代理时，"host_string=" 的域名/IP指向的是"反向代理服务"的IP和端口。协议头为"http://"或"https://"。
 #如果被反向代理映射到自定义目录，"host_string=" 要以"/"结尾。如: "https://ip:port/myjeppQuery/"

 #数据设置
 [flightplan]
 fltplan=HistoryFlightPlan.db
 ----ini配置文件----
 -------航线数据库的说明, 格式sqlite3, 共有4个table-------
CREATE TABLE flightPlan (
      flightPlanID integer primary key autoincrement,
      uid smallint unsigned,   -- (用户组uid)
      createtime datetime default '2016-9-27',
      routeid varchar(16),     -- 航路代码,如CANPEK01A
      departure varchar(16),   -- 起飞机场icao
      arrival varchar(16),     -- 落地机场icao
      alternate varchar(1024),  -- 逗号分隔,不要有空格
      route varchar(4096)       -- 空格分隔,标准航路格式,
      );
CREATE TABLE user (
      uid smallint unsigned not null primary key default 0,  -- 如需实现不同帐号相同uid,(需要去掉primary key)
                                                 -- uid必须 >=0; 其中 uid=0 为特殊用户,可以查询所有的航线数据.
      username varchar(32) not null default '',  -- 用户名 ,部分特殊字符可能不支持(:@?)
      password varchar(64) default ''            -- 密码
      );
CREATE TABLE icaoiata (  --程序中没有使用
      icao varchar(4),
      iata varchar(4)
      );
CREATE TABLE alternate(
      airport varchar(16),   -- 机场icao,单个机场,或机场对
      actype char(3),        -- 飞机型号,三字代码,使用纯数字, 如:319,738,787
      aircraft varchar(16),  -- 飞机详细型号,随便写,程序中没用到
      alternate varchar(1024)  -- 备降场icao,逗号分隔,不要有空格
      );
CREATE INDEX alternateidx on alternate(airport);
------示例数据,开始----
insert into user values(1,'test','test001');

insert into flightPlan values(1000,1,datetime('now','localtime'),'HELP','INFO','MESSAGE','This is a INFO msg.','If you have any suggestion or comments, PLS send email to osnosn@126.com');
insert into flightPlan values('1001','1','2022-10-14 20:28:55','CANPEK01','ZGGG','ZBAA','ZGSZ,ZGKL,ZJHK,ZHHH,ZSHC,ZYTX,ZYTL,ZBTJ,ZHCC','ZGGG YIN1A YIN A461 WXI A461 HG W81 BOBAK BOBA7A ZBAA');
insert into flightPlan values('1002','1','2022-10-14 20:28:55','PEKCAN01','ZBAA','ZGGG','ZYTX,ZYTL,ZBTJ,ZHCC,ZGSZ,ZGKL,ZJHK,ZHHH,ZSHC','ZBAA REN12D RENOB G212 LARAD B458 WXI A461 LIG R473 R473 R473 BEMAG V5 ATAGA ATA1A ZGGG');
insert into flightPlan values('1003','1','2022-10-14 20:28:55','CANCTU01','ZGGG','ZUUU','ZGSZ,ZGKL,ZJHK,ZHHH,ZSHC,ZUCK','ZGGG YIN1E YIN G586  QP B330 ELKAL W179 XYO W25 FJC FJC01A ZUUU');
insert into flightPlan values('1004','1','2022-10-14 20:28:55','CTUCAN01','ZUUU','ZGGG','ZUCK,ZGKL,ZGSZ,ZJHK,ZHHH,ZSHC','ZUUU ZYG01D ZYG B330 KWE W181 DUDIT A599 GYA GYA1A ZGGG');
insert into flightPlan values('1009','1','2022-10-14 20:28:55','CANSYD01','ZGGG','YSSY','VHHH,ZBAA,YMML,NZAA,AYPY,NWWW,NZCH,RPLL,RPMD,RPMZ,RPVM,VVTS,WAAA,WABB,WADD,WAMM,WBKK,WBSB,WIII,WSSS,YBAS,YBCS,YBRK,YBTL,YPAD,YPDN,YPEA,YPLM,YPPH,YPTN,YSCB,YSSY,RPLC','ZGGG VIB1G VIBOS R473 SIERA MULET SKATE V5 SABNO A583 ZAM A461 AMN R340 AGETA UH201 SCO CORKY SADLO BOREE BOREE6 YSSY');
insert into flightPlan values('1010','1','2022-10-14 20:28:55','SYDCAN01','YSSY','ZGGG','YMML,NZAA,VHHH,AYPY,NWWW,NZCH,RPLL,RPMD,RPMZ,RPVM,VVTS,WAAA,WABB,WADD,WAMM,WBKK,WBSB,WIII,WSSS,YBAS,YBCS,YBRK,YBTL,YPAD,YPDN,YPEA,YPLM,YPPH,YPTN,YSCB,YSSY,RPLC','YSSY DEENA6 RIC BOYSY MISIT MUDGI AGETA R340 AMN A461 ZAM A583 SABNO TOFEE SUKER ARROW J103 PICTA CH B330 TAMOT W68 IDUMA IDU81A ZGGG');

insert into icaoiata values('ICAO','IATA');
insert into icaoiata values('ZGGG','CAN');
insert into icaoiata values('ZUUU','CTU');
insert into icaoiata values('ZBAA','PEK');
insert into icaoiata values('YSSY','SYD');

insert into  alternate values('YSSY','330','A330-200','YMML,YBBN,YSCB,NZAA');
insert into  alternate values('ZBAA','737','A320','ZSJN,ZYTL,ZYTX,ZBTJ,ZHCC,ZSQD');
insert into  alternate values('ZBAA','319','A320','ZSJN,ZYTL,ZYTX,ZBTJ,ZHCC,ZSQD');
insert into  alternate values('ZBAA','320','A320','ZSJN,ZYTL,ZYTX,ZBTJ,ZHCC,ZSQD');
insert into  alternate values('ZGGG','320','A320','ZGSZ,ZGKL,ZJHK,ZSAM');
insert into  alternate values('ZUUU','320','A320','ZUCK,ZUGY,ZLXY,ZGKL');
insert into  alternate values('ZSSS','320','A320','ZSPD,ZSNJ,ZHHH,ZSAM,ZSQD');
insert into  alternate values('ZGGGYSSY','330','A330','NZAA,AYPY,NWWW,NZCH,RPLL,RPMD,RPMZ,RPVM,VVTS,WAAA,WABB,WADD,WAMM,WBKK,WBSB,WIII,WSSS,YBAS,YBCS,YBRK,YBTL,YPAD,YPDN,YPEA,YPLM,YPPH,YPTN,YSCB,YSSY,RPLC');  --按航线
insert into  alternate values('CANSYD01','330','A330','NZAA,AYPY,NWWW,NZCH,RPLL,RPMD,RPMZ,RPVM,VVTS,WAAA,WABB,WADD,WAMM,WBKK,WBSB,WIII,WSSS,YBAS,YBCS,YBRK,YBTL,YPAD,YPDN,YPEA,YPLM,YPPH,YPTN,YSCB,YSSY,RPLC');  --按航路代码
------示例数据,结束----
说明:
  查询航线, 调取指定航线 :
       如果指定机型,则通过alternate表查询备降场,忽略flightPlan表中的备降场。
          分别用 routeid, departure+arrival, departure, arrival 查询alternate表, 结果合并去重。
       如果没有指定机型,
          只通过 flightPlan表alternate字段,获取备降机场。
  jeppFD app 地图上显示的航路和备降场,以"调取指定航线"输出的内容为准。 
  download:  https://github.com/osnosn/jeppFlightPlan/ 
------JeppFD-Pro 或 FliteDeckPro 配置说明----
 创建好"航线数据库"，配置好 ini 文件，然后运行这个程序。
   ini 配置文件中 "address=" 就是程序监听的地址，用于配置ipad 的 URL。
 本程序只提供http服务。如需建立https服务，可以前置nginx或apache的https，然后反向代理/jepp/目录到本程序的监听地址和端口。Help页面和图片在/jeppS/目录。
 /jeppS/中的帮助页面可被替换。可以在ini所在目录，创建/jeppS/目录替换，仅支持.html .png .jpg .ico 文件，html为utf8编码。html 文件中支持"host_string="对"%HOST%"的简单替换.

   配置iPad的jeppFD的URL，就可以使用https连接，提高安全性。
 没有使用反向代理时，"host_string=" 的域名/IP指向的是"address="的IP和端口。协议头为"http://"。不要以"/"结尾。
 使用了反向代理时，"host_string=" 的域名/IP指向的是"反向代理服务"的IP和端口。协议头为"http://"或"https://"。
 如果被反向代理映射到自定义目录，"host_string=" 要以"/"结尾。如: "https://ip:port/myjeppQuery/"

 在 ipad 自带的系统设置 "设置" -> 在左侧找到 "FD Pro" 或 "FD Pro X" -> ACCOUNT INFO "Services"
  FLIGHT PLANS
    Username:   根据user表中的记录
    Password:   根据user表中的记录
    URL:  http://ipaddr:port/jepp/733738/     提供733和738的备降场列表
          http://ipaddr:port/jepp/330/        提供330的备降场列表
          http://ipaddr:port/jepp/319320321/    提供320系列的备降场列表
          http://ipaddr:port/jepp/            不考虑机型的备降场列表
          http://ipaddr:port/jepp/000/        不提供备降场列表
          http://ipaddr:port/jepp/id/         如果routeid有重名,使用flightPlanID调取航线
          https://domainName:port/jepp/000/   如果有前置https服务器
      注意，URL中,前后不能有空格。
  设置完成, 退出"设置".

  打开 FliteDeckPro , 
  找到 "Flight Plans" ,在"Call sign"位置,输入icao或iata机场对(不能包含空格)，就能看到航路计划了。
     比如: CANSYD 或者 ZGGGYSSY 
 -----
```
------
## nginx 配置样例
* 假设 jeppFlightPlan 监听在 127.0.0.1:7878
  ```
  [webserver]
  address=127.0.0.1:7878
  host_string=http://192.168.200.2/myjepp/
  ```
* nginx 工作在 192.168.200.2:80
  ```
  location /myjepp/ {
    proxy_http_version 1.1;
    proxy_pass http://127.0.0.1:7878/jepp/;
    proxy_set_header XForwarded-For $proxy_add_x_forwarded_for;
  }
  location /jeppS/ {
    proxy_http_version 1.1;
    proxy_pass http://127.0.0.1:7878;
    #or# proxy_pass http://127.0.0.1:7878/jeppS/;
    proxy_set_header XForwarded-For $proxy_add_x_forwarded_for;
  }
  ```
* 用浏览器访问 http://192.168.200.2/myjepp/ 就能看到说明。   
  网页的登录账号，用 db 的 user 表中的账号，test/test001   
