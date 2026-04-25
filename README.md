# M3U8 代理服务

## 1 项目说明

该项目可代理多种流媒体格式，包括：

- M3U8 文件（支持 **多级、多轨道** M3U8 文件）
- 视频文件
- 流式传输的视频
- **MPD 文件**

### 1.1 使用场景

- 某些客户端没有 IPV6 地址，可以将本项目部署到有 IPV6 地址的主机进行反代。
- 某些 URL 需要特定网络环境才能访问，可以统一在一台主机管理。

### 1.2 支持的格式

> 如需自定义 Content-Type，可修改文件中的代码：[route/consts/url_type.py](route/consts/url_type.py)

| 文件类型       | 支持的 Content-Type（正则表达式）                           |
| -------------- | ----------------------------------------------------------- |
| M3U8 文件      | 见文件 [route/consts/url_type.py](route/consts/url_type.py) |
| 视频文件       | `'application\\/octet-stream', '^video\\/.*$'`              |
| 流式传输的视频 | `'application\\/octet-stream', 'video\\/x-flv'`             |
| MPD 文件       | `'application\\/octet-stream', 'application/dash'`          |

## 2 项目启动

### 2.1 安装依赖

使用[uv](https://docs.astral.sh/uv/getting-started/installation)作为包管理器。

```shell
# 初始化虚环境
uv venv

# 激活虚环境
source .venv/bin/activate

# 安装依赖
uv sync
```

### 2.2 启动命令

从example复制一份配置文件，并重命名为固定值

```bash
cp config/m3u8-proxy-server-example.json config/m3u8-proxy-server.json
```

1. gunicorn启动

   > 注意：需要设置超时时间，否则在代理流式传输的视频时，会播放一段时间就自动停止。
   >
   > 设置`--timeout 600` 参数表示 10*60 秒内自动关闭连接

   ```sh
   # gunicorn --timeout 600 -w 线程数 -c gunicorn_config.py m3u8ProxyServer:app
   # 例如
   gunicorn --timeout 600 -w 4 -c gunicorn_config.py m3u8ProxyServer:app
   ```

2. 直接启动（单线程）

   ```sh
   # 无输出日志
   # python3 -u m3u8ProxyServer.py >> /dev/null 2>&1 &

   # 输出日志
   /usr/bin/python3 -u m3u8ProxyServer.py >> run.log 2>&1 &
   ```

3. 测试代理

    使用base64编码URL，可以访问[Online Base64](https://emn178.github.io/online-tools/base64_encode.html)选择`RFC4648 URLSafe`模式生成，也可以运行以下代码生成。

    ```python
    # 生成URL
    import base64
    url='https://xxx.com/123.m3u8'.encode()
    print(base64.urlsafe_b64encode(url).decode())
    # 输出 aHR0cHM6Ly94eHguY29tLzEyMy5tM3U4
    ```

    使用curl测试代理是否正常302跳转

    ```bash
    curl -v -L http://127.0.0.1:18080/proxy/url/aHR0cHM6Ly94eHguY29tLzEyMy5tM3U4
    ```

    最后，将地址粘贴到VLC播放器测试。

### 2.3 监控脚本

使用 `monitor/monitor.sh` 脚本启动。

   - 该脚本可以用于 crontab 监控运行检查服务运行是否正常。
   - 使用前需要修改脚本里的文件夹路径。

## 3 技术文档

- [接口文档](docs/接口文档.md)
- [配置文件说明](docs/配置文件说明.md)

### 4 其他说明

- **默认不代理本地地址(127.0.0.1/localhost)，如有需要请在配置文件添加以下规则：**

  ```json
  {
    "proxy": {
      "server": {
        "rules": {
          "127\\.0\\.0\\.1": "default",
          "localhost": "default"
        }
      }
    }
  }
  ```
