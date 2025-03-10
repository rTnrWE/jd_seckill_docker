# Jd_Seckill_Docker优化版

## 特别声明:
 
* 本仓库发布的`jd_seckill`项目中涉及的任何脚本，仅用于测试和学习研究，禁止用于商业用途，不能保证其合法性，准确性，完整性和有效性，请根据情况自行判断。

* 本项目内所有资源文件，禁止任何公众号、自媒体进行任何形式的转载、发布。

* `chinaarjun` 对任何脚本问题概不负责，包括但不限于由任何脚本错误导致的任何损失或损害.

* 间接使用脚本的任何用户，包括但不限于建立VPS或在某些行为违反国家/地区法律或相关法规的情况下进行传播, `chinaarjun` 对于由此引起的任何隐私泄漏或其他后果概不负责。

* 请勿将`jd_seckill`项目的任何内容用于商业或非法目的，否则后果自负。

* 如果任何单位或个人认为该项目的脚本可能涉嫌侵犯其权利，则应及时通知并提供身份证明，所有权证明，我们将在收到认证文件后删除相关脚本。

* 以任何方式查看此项目的人或直接或间接使用`jd_seckill`项目的任何脚本的使用者都应仔细阅读此声明。`chinaarjun` 保留随时更改或补充此免责声明的权利。一旦使用并复制了任何相关脚本或`jd_seckill`项目，则视为您已接受此免责声明。
  
* 您必须在下载后的24小时内从计算机或手机中完全删除以上内容。  
  
* 本项目遵循`GPL-3.0 License`协议，如果本特别声明与`GPL-3.0 License`协议有冲突之处，以本特别声明为准。

> ***您使用或者复制了本仓库且本人制作的任何代码或项目，则视为`已接受`此声明，请仔细阅读***  
> ***您在本声明未发出之时点使用或者复制了本仓库且本人制作的任何代码或项目且此时还在使用，则视为`已接受`此声明，请仔细阅读***

## 简介
通过我这段时间的使用（2020-12-12至2020-12-17），证实这个脚本确实能抢到茅台。我自己三个账号抢了四瓶，帮两个朋友抢了4瓶。
大家只要确认自己配置文件没有问题，Cookie没有失效，坚持下去总能成功的。

根据这段时间大家的反馈，除了茅台，其它不需要加购物车的商品也不能抢。具体原因还没有进行排查，应该是京东非茅台商品抢购流程发生了变化。  
为了避免耽误大家的时间，先不要抢购非茅台商品。  
等这个问题处理好了，会上线新版本。


## 暗中观察

根据12月14日以来抢茅台的日志分析，大胆推断再接再厉返回Json消息中`resultCode`与小白信用的关系。  
这里主要分析出现频率最高的`90016`和`90008`。  

### 样例JSON
```json
{'errorMessage': '很遗憾没有抢到，再接再厉哦。', 'orderId': 0, 'resultCode': 90016, 'skuId': 0, 'success': False},
{'errorMessage': '很遗憾没有抢到，再接再厉哦。', 'orderId': 0, 'resultCode': 90008, 'skuId': 0, 'success': False}
```

### 数据统计

| 案例 | 小白信用 | 90016  | 90008  | 抢到耗时 |
| ---- | -------- | ------ | ------ | -------- |
| 张三 | 63.8     | 59.63% | 40.37% | 暂未抢到 |
| 李四 | 92.9     | 72.05% | 27.94% | 4天      |
| 王五 | 99.6     | 75.70% | 24.29% | 暂未抢到 |
| 赵六 | 103.4    | 91.02% | 8.9%   | 2天      |

### 猜测
推测返回90008是京东的风控机制，代表这次请求直接失败，不参与抢购。  
小白信用越低越容易触发京东的风控。  

从数据来看小白信用与风控的关系大概每十分为一个等级，所以赵六基本上没有被拦截，李四和王五的拦截几率相近，张三的拦截几率最高。  

风控放行后才会进行抢购，这时候用的应该是水库计数模型，假设无法一次性拿到所有数据的情况下来尽量的做到抢购成功用户的均匀分布，这样就和概率相关了。  

> 综上，张三想成功有点困难，小白信用是100+的用户成功几率最大。

## 主要功能

- 登陆京东商城（[www.jd.com](http://www.jd.com/)）
  - 用京东APP扫码给出的二维码
- 预约茅台
  - 定时自动预约
- 秒杀预约后等待抢购
  - 定时开始自动抢购

## 运行环境

- [Python 3](https://www.python.org/)

## 第三方库

- 需要使用到的库已经放在requirements.txt，使用pip安装的可以使用指令  
`pip install -r requirements.txt`
- 如果国内安装第三方库比较慢，可以使用以下指令进行清华源加速
`pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/`

## 使用教程  
#### 1. 推荐Chrome浏览器
#### 2. 网页扫码登录，或者账号密码登录
#### 3. 填写config.ini配置信息 
(1)`eid`和`fp`找个普通商品随便下单,然后抓包就能看到,这两个值可以填固定的 
> 随便找一个商品下单，然后进入结算页面，打开浏览器的调试窗口，切换到控制台Tab页，在控制台中输入变量`_JdTdudfp`，即可从输出的Json中获取`eid`和`fp`。  
> 不会的话参考原作者的issue https://github.com/zhou-xiaojun/jd_mask/issues/22

(2)`sku_id`,`DEFAULT_USER_AGENT` 
> `sku_id`已经按照茅台的填好。
> `cookies_string` 现在已经不需要填写了
> `DEFAULT_USER_AGENT` 可以用默认的。谷歌浏览器也可以浏览器地址栏中输入about:version 查看`USER_AGENT`替换

(3)配置一下时间
> 现在不强制要求同步最新时间了，程序会自动同步京东时间
>> 但要是电脑时间快慢了好几个小时，最好还是同步一下吧

以上都是必须的.
> tips：
> 在程序开始运行后，会检测本地时间与京东服务器时间，输出的差值为本地时间-京东服务器时间，即-50为本地时间比京东服务器时间慢50ms。
> 本代码的执行的抢购时间以本地电脑/服务器时间为准

(4)修改抢购瓶数
> 代码中默认抢购瓶数为2，且无法在配置文件中修改
> 如果一个月内抢购过一瓶，最好修改抢购瓶数为1 
> 具体修改为：在`jd_spider_requests.py`文件中搜索`self.seckill_num = 2`，将`2`改为`1`

#### 4.运行main.py 
根据提示选择相应功能即可。如果出现请扫码登录的提示可查看项目目录下是否存在`qr_code.png`文件,若存在打开图片，并使用京东手机APP扫码登录即可。

- *Linux下命令行方式显示二维码（以Ubuntu为例）*

```bash
$ sudo apt-get install qrencode zbar-tools # 安装二维码解析和生成的工具，用于读取二维码并在命令行输出。
$ zbarimg qr_code.png > qrcode.txt && qrencode -r qrcode.txt -o - -t UTF8 # 解析二维码输出到命令行窗口。
```

#### 5.抢购结果确认 
抢购是否成功通常在程序开始的一分钟内可见分晓！  
搜索日志，出现“抢购成功，订单号xxxxx"，代表成功抢到了，务必半小时内支付订单！程序暂时不支持自动停止，需要手动STOP！  
若两分钟还未抢购成功，基本上就是没抢到！程序暂时不支持自动停止，需要手动STOP！  

Docker 运行
自行准备 docker，docker-compose 环境

编译项目

构建镜像
* cd dockerfile
* docker build jd-seckill:latest .
运行容器
修改配置文件 compose/common/configs/config.ini

使用 Docker compose 运行

* cd compose
* docker-compose up -d # -d 后台运行。
默认运行选项为秒杀
如果构建镜像名不是 jd-seckill:latest 你需要修改 docker-compose.yml 中的镜像。
查看运行状态
# 确认 State 为 UP。
$ docker-compose ps
# 查看并跟踪运行日志。
$ docker logs jd-seckill -f
登录账号
执行命令输出二维码扫码登录

$ docker exec jd-seckill jd-seckill qrcode
更多
# 秒杀预约
$ docker exec jd-seckill jd-seckill reserve
# 执行秒杀
$ docker exec jd-seckill jd-seckill seckill

## 直接使用已编译好的镜像运行 chinaarjun/jd-seckill-docker 

### docker-compose运行
* 运行容器
* 修改配置文件 compose/common/configs/config.ini
* 在compose目录中执行命令 docker-compose up -d
* 注意：如果运行失败请核对docker-compose.yaml文件中镜像名称是否正确

### docker命令运行，只需要配置config.ini
* 执行命令: 
``` shell
docker run -v /home/docker/jd_seckill/config.ini:/app/config.ini -d --name jd-seckill chinaarjun/jd-seckill-docker 
```
* /home/docker/jd_seckill/config.ini 修改为本地绝对路径
* -d #默认后台运行


## 喜欢记得Star谢谢


## 打赏
要是客官抢到了茅台，心情好，请我喝一杯咖啡好不好:)  
![收款二维码](https://github.com/ChinaArJun/jd_seckill_docker/blob/main/wx.jpg)
>>>>>>> master
