## docker 練習筆記

### docker 指令

啟動Continer，使用以下的指令執行如下
````bash
docker run -d -p 8080:80 --restart=always --name nginx nginx
````
參數說明：
* -d：把 container 執行在背景裡
* -p: 做 port 的mapping，container裡的port 80 mapping 到 host 的8080 port
* --restart=always：如果 container 遇到例外的情況被 stop 掉，例如是重新開機，docker 會試著重新啟動此 container
* --name=registry：設定 container 的 name 為 nginx
* 最後一個參數 nginx 是 docker image 的 Name


如何看到執行了哪些 Container，可以使用以下的指令
```bash
docker ps -a
```
參數說明：
* -a：如果沒有加上 -a 參數，只會顯示 running 的 container

在執行 Container 如何看到 log，指令如下
```bash
docker logs nginx
```

如何把執行的 Container 刪除掉，使用以下的指令
```bash
docker rm -f nginx
```
參數說明：
* -f：強制刪除 Container
* nginx 為 Container Name

如何把 Docker Image 刪除掉，使用以下的指令
```bash
docker rmi docker.io/nginx
```
參數說明：
* docker.io/nginx 為在 pull Docker Image 的 name，其實可以把 docker.io 省略掉

如果要進入 container 看有哪些資料夾、檔案或是要修改檔案可以使用以下的指令：
```bash
docker exec -it nginx /bin/bash
```
* -i：Keep STDIN open even if not attached，讓標準輸入維持在打開的狀態
* -t：Allocate a pseudo-tty，替Container配置一個虛擬的終端機
有了 `-it` 這樣這個Container就擁有了標準的輸入輸出能力
* nginx 為 Container Name，也可以使用 Container ID

進入 container 操作資料的方式有很多，但是 Docker 不建議直接使用 ssh 連到 Container 裡面去操作資料，主要是考量到安全性和每一個 container 都是一個process，所以不希望使用 ssh 直接進入 Container 裡面，應該要使用 Docker 提供的 docker exec 指令。


停止 Container 的執行可以使用以下的指令
```bash
docker stop nginx
```

Docker Container 被 stop 停止時，可以使用 docker start 指令把 Container重新啟動起來，指令如下
```bash
docker start nginx
```

在 host 和 docker 之間傳遞檔案的指令如下
```bash
docker cp /path/to/file1 DOCKER_ID:/path/to/file2
```


### Dockerfile
建立 Dockerfile 可以讓我們使用程式化的方式設定好 docker 所需的功能或服務，再透過 `docker build` 的指令就可以把 docker image 建構起來，後續要使用時就直接 run 此 image。   
下面建立一個 python pyppeteer 的爬蟲，由 flask 作為 api 控制。由於會使用到 headless browser，因此也在建立 image 時事先裝好。  


#### 一. 建立 Dockerfile
建立寫 Dockerfile 會用到的資料夾，並撰寫

```bash
mkdir docker-test
cd docker-test
vim Dockerfile
```

Dockerfile 的內容如下
```sh
# Use the official Python image.
FROM python:3.8-slim

MAINTAINER hotaiconnected Jerry.Ko

# Install manually all the missing libraries
RUN apt-get update
RUN apt-get install -y \
	pip \
	gconf-service \
	libasound2 \
	libatk1.0-0 \
	libcairo2 \
	libcups2 \
	libfontconfig1 \
	libgdk-pixbuf2.0-0 \
	libgtk-3-0 \
	libnspr4 \
	libpango-1.0-0 \
	libxss1 \
	fonts-liberation \
	libnss3 \
	lsb-release \
	xdg-utils
RUN apt-get install -y libappindicator1; apt-get -fy install

# Install Python dependencies.
COPY requirements.txt \tmp\requirements.txt
RUN pip install -r \tmp\requirements.txt

COPY init_pyppeteer.py \tmp\init_pyppeteer.py
RUN python3 \tmp\init_pyppeteer.py
COPY main.py main.py

EXPOSE 5000

CMD python main.py
```
以上的 Dockerfile 主要有用到的指令說明如下
* FROM：使用到的 Docker Image 名稱，今天使用 python:3.8-slim

* MAINTAINER：用來說明，撰寫和維護這個 Dockerfile 的人是誰，也可以給 E-mail的資訊

* RUN：RUN 指令後面放 Linux 指令，用來執行安裝和設定這個 Image 需要的東西

* COPY：把 Local 的檔案複製到 Image 裡

* EXPOSE：開啟 container 的 port 接口

* CMD：在指行 docker run 的指令時會直接呼叫開啟 python 

#### 二. Build Docker Image
預設在和 Dockerfile 檔案同層的資料夾底下輸入， docker build 指令，如下
```bash
docker build -t my-crawler . --no-cache
```
使用 `--no-cache` 的主要原因，是避免在 Build Docker image 時被 cache 住，而造成沒有 build 到修改過的 Dockerfile。  
Build 完的結果如下圖：
![build_docker_success](./plot/build_docker_success.png)
最後看到 `Successfully tagged my-crawler:latest` 後，代表 build 完成。  
可使用 docker images 指令查看是否有 build 成功。


#### 三. 啟動 Docker Container，這時 flask 的 service 也會被執行起來
```bash
docker run -p 5000:5000 my-crawler
```
flask service 會被執行起來的主要原因是在 Dockerfile 裡面有寫 CMD 指令，呼叫啟動 flask service。


#### 四. 使用其他主機的 Browser 確認
直接使用 docker container 所在的主機 ip 位置，即可回吐爬蟲程式執行的結果。
![exce_crawler](./plot/exce_crawler.png)