# Docker配置es的ingest-attachment插件

## 1.离线下载

1. 首先根据这个链接下载zip文件：

   - https://artifacts.elastic.co/downloads/elasticsearch-plugins/ingest-attachment/ingest-attachment-7.12.1.zip

2. 将文件解压到目录`/usr/local/dk/docker/elasticsearch/plugins/`中并且命名为

3. ingest-attachment![image-20230304211137713](E:/Development/Typora/images/image-20230304211137713.png)

4. #将文件夹拷贝到elasticsearch容器中

   - ```
     docker cp ingest-attachment es:/usr/share/elasticsearch/plugins
     ```

5. 重启es

   - ```bash
     docker restart es
     ```

6. 查看插件是否安装成功

   [106.53.124.182:9200/_cat/plugins](http://106.53.124.182:9200/_cat/plugins)

![image-20230304211510705](E:/Development/Typora/images/image-20230304211510705.png)





![image-20230304211624938](E:/Development/Typora/images/image-20230304211624938.png)