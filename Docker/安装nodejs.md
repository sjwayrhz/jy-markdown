# Install node.js

## Install Node.js and npm from NodeSource repository

The simplest way to install Node.js and npm is from the **NodeSource repository**.

1. First, update the local repository to ensure you install the  latest versions of Node.js and npm. Type in the following command:

```
sudo yum update
```

2. Next, add the NodeSource repository to the system with:

```
curl –sL https://rpm.nodesource.com/setup_10.x | sudo bash -
```

3. The output will prompt you to use the following command if you want to install Node.js and npm:

```
sudo yum install nodejs
```

4. Finally, verify the installed software with the commands:

```
npm -v
node -v
```
安装umi

```
npm install -g umi --registry=https://registry.npm.taobao.org -g
```

以下方法使用 yarn，不推荐

----------------------------

安装cnmp

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
```
 cnpm -v
```
安装tyarn
```ruby
cnpm i yarn tyarn -g
```
```ruby
tyarn -v
```
安装umi

```shell
tyarn global add umi
umi -v
```
安装静态服务

```shell
cnpm install -g serve
```
安装插件
```
tyarn add umi-plugin-react --dev
```

yum install  npm 
npm install -g n
n v10.16.0