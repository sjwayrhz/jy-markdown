项目克隆
```
git clone git@gitlab.juneyaokc.com:group-service/group-app-manage.git
```
安装编译
```
npm install --registry https://registry.npm.taobao.org --unsafe-perm
npm run build
```

Dockerfile
```
FROM node:14.17.5

WORKDIR /vue
RUN rm -rf node_modules
COPY . .

RUN npm i --registry=http://registry.npm.taobao.org --unsafe-perm --unsafe-perm

RUN npm run build

FROM nginx:alpine

COPY nginx.vue.conf /etc/nginx/conf.d/nginx.vue.conf
RUN mkdir -p /usr/local/nginx/html
COPY --from=0 /vue/dist /usr/share/nginx/html

EXPOSE 8082
ENTRYPOINT nginx -g 'daemon off;' 

```

nginx.vue.conf
```
server {
  listen  8082;
  server_name localhost;

  location / {
    root /usr/share/nginx/html; 
    index  index.html;
  }

  location /juneyao {
    proxy_pass http://dev-web.windfindtech.com;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Host $host;
  }
}
```