
> `设备终端通信协议` 是指定位测量设备与手持终端之间的点对点文本传输协议。一般情况下，需要同时支持蓝牙、串口等两种主要的通信方式。

### 通用格式标准
- 起始符：`$`
- 消息类型码：3~8 字母
- 字段分隔：`,`（逗号）
- 校验分隔符：`*` 后跟两位十六进制校验（大写），例如 `*4A`
- 结束符：`\r\n`
- 消息时间戳：UTC 时间 hhmmss.ss
- 校验算法：**XOR** 校验（对 `$` 与 `*` 之间的所有字节做按位异或）
- 编码：UTF-8（但实际字段均为 ASCII）
- 字符串长度：单条消息不应超过 2048 字节（串口缓冲/串行稳定性考虑）
- 时间同步：多个设备数据融合时，以卫星定位时间为基准时间

### 通信类型及用途
- **蓝牙通信**     软件接口的标准通信方式
- **串口通信**     用于设备的初始设置与输入/输出调试
- WIFI              传输 RTMP/RTSP 视频流


指令

```bash
## 指令格式(请求/应答, HH 表示2位十六进制校验吗)
$CMD,subcommand param1 ... paramN*HH\r\n
# 正常应答
$ACK,subcommand param1 ... paramN,:OK*HH\r\n
# 错误应答（...,:错误消息*HH\r\n）
$ACK,subcommand param1 ... paramN,:PARSING FAILD ...*HH\r\n

# 设备电源信息输出配置
$CMD,DEV.CONFIG POWER 1s*HH\r\n

## GNSS指令
# 配置设备端 GNSS 串口
$CMD,DEV.CONFIG GNSS COM1 115200*HH\r\n
# 配置 GNSS 报文输出频率 （GN--- 为多星模式，单北斗可以是：BD---）
$CMD,DEV.CONFIG GNSS.GNGGA 1hz*HH\r\n
$CMD,DEV.CONFIG GNSS.GNGSV 1hz*HH\r\n
$CMD,DEV.CINFIG GNSS.GNGSA 1hz*HH\r\n
$CMD,DEV.CONFIG GNSS.GNRMC 1hz*HH\r\n
$CMD,DEV.CONFIG GNSS.GNHDT 1hz*HH\r\n
# --HPD 属于NMEA的扩展报文，可以是单天线GNSS + IMU的融合数据
$CMD,DEV.CONFIG GNSS.GNHPD 1hz*HH\r\n
# 启动（ID 可选）/关闭
$CMD,DEV.CTRL GNSS.OPEN ID*HH\r\n
$ACK,DEV.CTRL GNSS.OPEN ID,:OK*HH\r\n
$CMD,DEV.CTRL GNSS.CLOSE ID*HH\r\n

## IMU指令
# IMU在设备端的输出频率
$CMD,DEV.CONFIG IMU 200hz*HH\r\n
# IMU通信传输频率
$CMD,DEV.CONFIG IMU.LOG 1hz*HH\r\n
# 启动/关闭
$CMD,DEV.CTRL IMU.OPEN ID*HH\r\n
$CMD,DEV.CTRL IMU.CLOSE ID*HH\r\n

## 激光测距指令
# 测距
$CMD,DEV.CONFIG LASER.LRG 1hz*HH\r\n
# 定位
$CMD,DEV.CONFIG LASER.LPO 1hz*HH\r\n
$CMD,DEV.CTRL LASER.OPEN ID ON*HH\r\n
$CMD,DEV.CTRL LASER.CLOSE ID ON*HH\r\n

## 相机指令(ID必须指定，用来区分双摄)
# AUTH_BASE64是WIFI登陆凭证，原始格式是 ssid:pwd，省略则为查询设备联网信息
$CMD,DEV.CONFIG CAMERA.NETWORK AUTH_BASE64*HH\r\n
# 应答包含局域网IP，网关地址，MAC地址（移动终端须检测是否与设备在同一网络内）
$ACK,DEV.CONFIG CAMERA.NETWORK ...,:OK LAN_IP LAN_GATEWAY MAC_ADDR*HH\r\n
# 另一种响应格式
$ACK,DEV.CONFIG CAMERA.NETWORK ...,:OK LAN_IP=xx;LAN_GATEWAY=xx;...;*HH\r\n
$CMD,DEV.CTRL CAMERA.OPEN ID*HH\r\n
# 使用指令打开摄像头时，将返回当前视频流格式以及URL
# ID：相机编号，LABEL：相机名称，W H:视频分辨率，FPS：帧率，ENC：MJPEG/H264/RAW
# URL示例: rtmp://192.168.1.2:8554/live1
$ACK,DEV.CTRL CAMERA.OPEN ID,:LAB=;W=;H=;FPS=;ENC=;URL=;*HH\r\n
$CMD,DEV.CONFIG CAMERA.CLOSE ID*HH\r\n

```

### 数据消息

```bash
## 设备电量消息
# SOURCE：BAT1/BAT2/MAIN 主电池/备用电池/外部供电
# VOLT_CURR：当前电压（V）
# VOLT_MIN：电压下限报警
# VOLT_MAX：最大
# SOC：剩余百分比（0-100）
# CHG_STAT：C=充电, D=放电, I=静止
# TEMP：电池温度（°C）
$PWR,UTIME,SOURCE,VOLT_CURR,VOLT_MIN,VOLT_MAX,SOC,CHG_STAT,TEMP*HH\r\n

# GNSS 数据消息将引用 NMEA 标准协议
$GNGGA,UTIME,...*HH\r\n
$GNGSV,UTIME,...*HH\r\n

# HPD 位置与姿态
# GPS_WEEK 1980-1-6至当前的星期数 xxxx
# GPS_SEC 当日0点起的秒 xxxx.xx
# HEADING GNSS 航向，PITCH 俯仰，ROLL 横滚，LAT 纬度，LON 精度，ALT 高程
# DX 基线东向距离，DY 基线北向距离，DZ 基线天向距离
# VX 东向速度，VY 北向速度，VZ 天向速度
# VDX 两次测量值间的东向速度差
# VDY 两次测量值间的北向速度差
# VDZ 两次测量值间的天向速度差
# BASE 基线长度
# STAT 0:GPS 无效；1:GPS 单点解；2:伪距差分；4:RTK 固定解；5:RTK 浮点解 
$GNHPD,GPS_WEEK,GPS_SEC,HEADING,
  PITCH,ROLL,LAT,LON,ALT,DX,DY,DZ,VX,VY,VZ,VDX,VDY,VDZ,BASE,STAT,*HH\r\n

# IMU 数据消息 (这里只考虑输出IMU的姿态计算结果)
$IMU,UTIME,ROLL,PITCH,YAW,STAT*HH\r\n

# 激光测距 
# DIST: 距离，UNIT：M，STRENGTH： 回波强度（可选），STAT：有效性（0：无效，1: 有效）
$LRG,UTIME,DIST,UNIT,STRENGTH,STAT*HH\r\n
# 激光定位结果（QUAL 评价值）
$LPO,UTIME,POS_X,POS_Y,POS_Z,ROLL,PITCH,YAW,QUAL*HH\r\n
```












