## Dockerizing a Node.js web app

這個範例的目的是要告訴你如何在 Docker container 上運行 Node.js 應用程式。這份說明文件適用於開發，而不是生產佈署。並且假設你已經有一個可以正常運作的[Docker 環境](https://docs.docker.com/engine/installation/) 與對於如何建構一個 Node.js 應用程式有了基本的了解。

首先，我們使用 Node.js 建立一個簡單的 web 應用程式，接下來我們為這個應用程式建立一個 Docker image ，最後我們在 container 執行這個 image。

Docker 允許你將應用程式與相關套件打包並轉變成標準化的元件稱為 container ，提供軟體開發。
> - container 是一個 stripped-to-basics 版本的 linux 作業系統。
> - image 是一個讓你載入到 container 的軟體。

##Create the Node.js app

首先，建立一個目錄給所有檔案存取，在這個目錄建立一個 `package.json` 檔案，這個檔案描述你的應用與記錄相關的套件。

```
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "First Last <first.last@example.com>",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.13.3"
  }
}
```

然後，建立一個 `server.js` 檔案來定義 web app 所使用的 [Express.js](http://expressjs.com/) framework：

```
'use strict';

const express = require('express');

// Constants
const PORT = 8080;

// App
const app = express();
app.get('/', function (req, res) {
  res.send('Hello world\n');
});

app.listen(PORT);
console.log('Running on http://localhost:' + PORT);
```

接下來的步驟，我們將探討如何在 Docker container 中使用官方的 Docker image 執行這個 app。首先你需要為你的 app 建立一個 Docker image。

##Creating a Dockerfile

建立一個名字是 `Dockerfile` 的空檔案：

```
touch Dockerfile
```

使用你最喜歡的編輯器開啟 `Dockerfile`

一開始，我們需定義所要建立的 image，這部份我們將從 [Docker Hub](https://hub.docker.com/) 取得 `node` 並使用最新的 LTS (long term support) 版本。

```
FROM node:argon
```

下一步，我們建立一個目錄讓我們的應用程式保存於 image 裡，這將會是你應用程式執行的目錄：

```
# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
```

這個 image 已經安裝 Node.js 與 NPM，接下來我們需要使用 `npm` 安裝你應用程式相關的套件：

```
# Install app dependencies
COPY package.json /usr/src/app/
RUN npm install
```

使用 `COPY` 指令來包裝你在 Docker image 裡的應用程式原始碼：

```
# Bundle app source
COPY . /usr/src/app
```

你的應用程式綁定在 `8080` port ，所以你需要使用 `EXPOSE` 指令將它對應到 `docker` daemon：

```
EXPOSE 8080
```

最後但並非最不重要，使用定義執行時間的 `CMD` 指令來定義執行你的應用程式。這裡我們將使用基本的 `npm start` 透過執行 `node server.js` 來啟動你的伺服器：

```
CMD [ "npm", "start" ]
```

你的 `Dockerfile` 現在看起來應該像這樣：

```
FROM node:argon

# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Install app dependencies
COPY package.json /usr/src/app/
RUN npm install

# Bundle app source
COPY . /usr/src/app

EXPOSE 8080
CMD [ "npm", "start" ]
```

##Building your image
將目錄移至 Dockerfile 存在的目錄並且輸入下列命令建置你的 Docker image。 透過 `-t` 旗標標記你的 image 可以讓你往後更容易找到這個 image。使用 docker images 命令如下：

```
$ docker build -t <your username>/node-web-app .
```

現在，你可以透過 `docker images` 命令在 Docker 列表看到你所建立的 image ：

```
$ docker images

# Example
REPOSITORY                      TAG        ID              CREATED
node                            argon      539c0211cd76    3 weeks ago
<your username>/node-web-app    latest     d64d3505b0d2    1 minute ago
```

##Run the image

在獨立 container 中使用 `-d` 可以讓你的 image 執行於背景模式下。 `-p` 旗標可以重新對應 public port 與 private port。使用以下命令執行你之前所建置的 image：

```
$ docker run -p 49160:8080 -d <your username>/node-web-app
```

印出你 app 的輸出：

```
# Get container ID
$ docker ps

# Print app output
$ docker logs <container id>

# Example
Running on http://localhost:8080
```

如果你想要進入 container ，你可以使用 `exec` 命令：

```
# Enter the container
$ docker exec -it <container id> /bin/bash
```

##Test


若要測試你的 app，首先，必須先取得你 app 對應 Docker 的 port：

```
$ docker ps

# Example
ID            IMAGE                                COMMAND    ...   PORTS
ecce33b30ebf  <your username>/node-web-app:latest  npm start  ...   49160->8080
```

> 在上面的範例中，Docker container 的 8080 port 對應你機器的 49160 port。


現在，你可以使用 `curl` 呼叫你的 app (如果沒有安裝 `curl` ，可以透過 `sudo apt-get install curl` 安裝)：

```
$ curl -i localhost:49160

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 12
Date: Sun, 02 Jun 2013 03:53:22 GMT
Connection: keep-alive

Hello world
```

希望這個教學可以幫你在 Docker 建置與執行一個簡單的 Node.js 應用程式。

你可以在下列連結中找到更多關於 Docker 與在 Docker 運行 Node.js 的資訊：

- [Official Node.js Docker Image](https://registry.hub.docker.com/_/node/)
- [Official Docker documentation](https://docs.docker.com/)
- [Docker Tag on StackOverflow](http://stackoverflow.com/questions/tagged/docker)
- [Docker Subreddit](https://www.reddit.com/r/docker)

##參考
[NodeJS 官方原文](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
> 本篇是翻譯 NodeJS 官方文件