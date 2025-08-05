# WBO

WBO is an online collaborative whiteboard that allows many users to draw simultaneously on a large virtual board.
The board is updated in real time for all connected users, and its state is always persisted. It can be used for many different purposes, including art, entertainment, design, teaching.

A demonstration server is available at [wbo.ophir.dev](https://wbo.ophir.dev)

## Screenshots

<table>
 <tr>
  <td> The <i><a href="https://wbo.ophir.dev/boards/anonymous">anonymous</a></i> board
  <td> <img width="300" src="https://user-images.githubusercontent.com/552629/59885574-06e02b80-93bc-11e9-9150-0670a1c5d4f3.png">
  <td> collaborative diagram editing
  <td> <img alt="Screenshot of WBO's user interface: architecture" width="300" src="https://user-images.githubusercontent.com/552629/59915054-07101380-941c-11e9-97c9-4980f50d302a.png" />
  
  <tr>
   <td> teaching math on <b>WBO</b>
   <td> <img alt="wbo teaching" width="300" src="https://user-images.githubusercontent.com/552629/59915737-a386e580-941d-11e9-81ff-db9e37f140db.png" />
   <td> drawing art
   <td> <img alt="kawai cats on WBO" width="300" src="https://user-images.githubusercontent.com/552629/120919822-dc2c3200-c6bb-11eb-94cd-57a4254fbe0a.png"/>
</table>

## Running your own instance of WBO

If you have your own web server, and want to run a private instance of WBO on it, you can. It should be very easy to get it running on your own server.

### Running the code in a container (safer)

If you use the [docker](https://www.docker.com/) containerization service, you can easily run WBO as a container.
An official docker image for WBO is hosted on dockerhub as [`lovasoa/wbo`](https://hub.docker.com/r/lovasoa/wbo): [![WBO 1M docker pulls](https://img.shields.io/docker/pulls/lovasoa/wbo?style=flat)](https://hub.docker.com/repository/docker/lovasoa/wbo).

You can run the following bash command to launch WBO on port 5001, while persisting the boards outside of docker:

```bash
mkdir wbo-boards # Create a directory that will contain your whiteboards
chown -R 1000:1000 wbo-boards # Make this directory accessible to WBO
docker run -it --publish 5001:80 --volume "$(pwd)/wbo-boards:/opt/app/server-data" lovasoa/wbo:latest # run wbo
```

You can then access WBO at `http://localhost:5001`.

### Running the code without a container

Alternatively, you can run the code with [node.js](https://nodejs.org/) directly, without docker.

First, download the sources:

```
git clone https://github.com/lovasoa/whitebophir.git
cd whitebophir
```

Then [install node.js](https://nodejs.org/en/download/) (v10.0 or superior)
if you don't have it already, then install WBO's dependencies:

```
npm install --production
```

Finally, you can start the server:

```
PORT=5001 npm start
```

This will run WBO directly on your machine, on port 5001, without any isolation from the other services. You can also use an invokation like

```
PORT=5001 HOST=127.0.0.1 npm start
```

to make whitebophir only listen on the loopback device. This is useful if you want to put whitebophir behind a reverse proxy.

### Running WBO on a subfolder

By default, WBO launches its own web server and serves all of its content at the root of the server (on `/`).
If you want to make the server accessible with a different path like `https://your.domain.com/wbo/` you have to setup a reverse proxy.
See instructions on our Wiki about [how to setup a reverse proxy for WBO](https://github.com/lovasoa/whitebophir/wiki/Setup-behind-Reverse-Proxies).

## Translations

WBO is available in multiple languages. The translations are stored in [`server/translations.json`](./server/translations.json).
If you feel like contributing to this collaborative project, you can [translate WBO into your own language](https://github.com/lovasoa/whitebophir/wiki/How-to-translate-WBO-into-your-own-language).

## Authentication

WBO supports authentication using [Json Web Tokens](https://jwt.io/introduction). This should be passed in as a query with the key `token`, eg, `http://myboard.com/boards/test?token={token}`

The `AUTH_SECRET_KEY` variable in [`configuration.js`](./server/configuration.js) should be filled with the secret key for the JWT.

Within the payload, you can declare the user's roles as an array.
Currently the only accepted roles are `moderator` and `editor`.

- `moderator` will give the user an additional tool to wipe all data from the board. To declare this role, see the example below.
- `editor` will give the user the ability to edit the board. This is the default role for all users.

```json
{
  "iat": 1516239022,
  "exp": 1516298489,
  "roles": ["moderator"]
}
```

Moderators have access to the Clear tool, which will wipe all content from the board.

## Board name verification in the JWT

WBO supports verification of the board with a JWT.

To check for a valid board name just add the board name to the role with a ":". With this you can set a moderator for a specific board.

```json
{
  "roles": [
    "moderator:<boardName1>",
    "moderator:<boardName2>",
    "editor:<boardName3>",
    "editor:<boardName4>"
  ]
}
```

eg, `http://myboard.com/boards/mySecretBoardName?token={token}`

```json
{
  "iat": 1516239022,
  "exp": 1516298489,
  "roles": ["moderator:mySecretBoardName"]
}
```

You can now be sure that only users who have the correct token have access to the board with the specific name.

## Configuration

When you start a WBO server, it loads its configuration from several environment variables.
You can see a list of these variables in [`configuration.js`](./server/configuration.js).
Some important environment variables are :

- `WBO_HISTORY_DIR` : configures the directory where the boards are saved. Defaults to `./server-data/`.
- `WBO_MAX_EMIT_COUNT` : the maximum number of messages that a client can send per unit of time. Increase this value if you want smoother drawings, at the expense of being susceptible to denial of service attacks if your server does not have enough processing power. By default, the units of this quantity are messages per 4 seconds, and the default value is `192`.
- `AUTH_SECRET_KEY` : If you would like to authenticate your boards using jwt, this declares the secret key.

## Troubleshooting

If you experience an issue or want to propose a new feature in WBO, please [open a github issue](https://github.com/lovasoa/whitebophir/issues/new).

## Monitoring

If you are self-hosting a WBO instance, you may want to monitor its load,
the number of connected users, and various other metrics.

You can start WBO with the `STATSD_URL` environment variable to send it to a statsd-compatible
metrics collection agent.

Example: `docker run -e STATSD_URL=udp://127.0.0.1:8125 lovasoa/wbo`.

- If you use **prometheus**, you can collect the metrics with [statsd-exporter](https://hub.docker.com/r/prom/statsd-exporter).
- If you use **datadog**, you can collect the metrics with [dogstatsd](https://docs.datadoghq.com/developers/dogstatsd).

## Download SVG preview

To download a preview of a board in SVG format you can got to `/preview/{boardName}`, e.g. change https://wbo.ophir.dev/board/anonymous to https://wbo.ophir.dev/preview/anonymous. The renderer is not 100% faithful, but it's often good enough.
## 解决哪些痛点？
   归根结底，WBO 帮你搞定这些糟心事儿：• 远程头脑风暴：大家各自涂涂画画，想法秒显现，灵感不断穿插• 在线教学：老师可以划重点、写公式、演示示意图，学生还能实时互动• 团队原型设计：产品经理、设计师直接一起在线画线框图，省去反复截图发群的烦恼• 创意涂鸦：随手画个流程图、人物草图，灵感来了就开画
想要私有化部署？也非常简单，我来给你划个重点表格：
### 部署方式
     一. 步骤概要Docker 容器（推荐）：隔离、安全、快速启动
     1. mkdir wbo-boards
     2. chown -R 1000:1000 wbo-boards
     3. docker run -it -p 5001:80 -v $(pwd)/wbo-boards:/opt/app/server-data lovasoa/wbo:latest
    二.  Node.js ：直跑零依赖容器，方便调试
     1. git clone https://github.com/lovasoa/whitebophir.git
     2. npm install --production
     3. PORT=5001 npm start
    三.子目录部署（反向代理）可集成到既有域名下的子路径，地址更“官方化”配置 Nginx/Apache，参见 Wiki 反向代理教程
### 安装完之后，浏览器打开 http://你的域名:5001，点“New board”就能开始画。想分享？把链接丢给同事，1、2 秒钟进来一起涂鸦。
### 主要功能&亮点功能
    模块说明画笔&形状工具：支持多种颜色、直线、矩形、椭圆、箭头，基础绘图齐活
    文本&涂鸦：文字输入、图像粘贴，写笔记、贴小纸条都没问题
    实时同步：任意人数同时在线，所有人都能看到同一块白板上的变化
    自动持久化存储：板面状态保存在服务器，断线重连或者关浏览器都能马上回来
    多语言&扩展性：支持中文等多种界面语言，背后完全开源、想改就改
    JWT 认证&权限控制：支持 Json Web Token，能给不同用户分配 editor/moderator 角色
### 优缺点一览
    优点：免费+开源，支持自建部署
          完全无广告、无内置付费墙
          部署简单，支持 Docker，超快上手
          支持离线断线重连；自动保存
          社区活跃，有不少贡献者和丰富扩展
    缺点：
          高级功能（比如音视频）需要另选其他工具
          UI 比商业产品简陋，缺少“花里胡哨”
          大规模会议时对服务器性能有一定要求
          内置聊天/语音功能不足，需要配合第三方工具
          SVG 导出偶尔渲染不完美
### 小结
     要说蝴蝶效应的源头可能就是白板上的一笔灵光乍现。WBO 的出现，正好填补了一个低成本、高效率、可自建的“在线画板”市场空白。无论是远程开会、在线教学 
     还是产品原型设计，你都能拿出它来给团队加速。再也不用在微信群里疯狂截图、发屁颠屁颠下载客户端了，打开网页就能一起画。
### 项目地址： https://github.com/lovasoa/whitebophir
