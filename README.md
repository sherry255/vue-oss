# vue_oss
![demo](http://i4.piimg.com/1949/ecb80f37a321a1dd.gif)

> oos upload 


## demo : 
地址: http://sherrycherish.cc/vue-oss/
### 因为cors的问题, 必须按照以下操作, demo才能完整展示:
 1. 安装插件: https://chrome.google.com/webstore/detail/allow-control-allow-origi/nlfbmbojpeacfghkpbjhddihlkkiljbi/reviews?hl=zh-CN
 2. ![启用cors](http://i2.muimg.com/1949/e1cb3540d166cfb6.png)
 
 #### 注: 使用完毕记得关闭cors
 
 



## Build Setup

``` bash

# 安装前准备
1. 更换淘宝源:

npm install -g cnpm --registry=https://registry.npm.taobao.org

2. 检查包的更新: 

cnpm i -g npm-check-updates
npm-check-updates -u 
# 安装
cnpm install


# 修改你的接口:
参考服务端签名后直传文档:https://help.aliyun.com/document_detail/31926.html
将此处的 serverUrl 换成你的接口地址: https://github.com/sherrycherish/vue-oss/blob/master/src/components/Upload.vue#L73



# serve with hot reload at localhost:8080
npm run dev


# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run e2e tests
npm run e2e

# run all tests
npm test
```




