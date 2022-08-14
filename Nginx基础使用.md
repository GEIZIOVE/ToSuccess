# Nginx基础使用

## 配置nginx

```nginx
server {

   listen 80;

   server_name localhost;

   location /api/hosp/ {      

      proxy_pass http://localhost:8201;

   }

   location /api/cmn/ {      

      proxy_pass http://localhost:8205;

   }
...
}
```



## 详细使用

[(82条消息) nginx（一） nginx详解_尐譽的博客-CSDN博客_nginx详解](https://blog.csdn.net/tjiyu/article/details/53027619)