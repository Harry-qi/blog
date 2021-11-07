---
title: Puppeteer生成pdf
date: 2020-12-04 11:10:49
tags:
---
> 最近公司业务需要打印pdf的功能，先说需求：
> - 封面(就是第一页内容)在web端不用展示，打印的pdf需要展示封面。 
> - 内容最多会有6页。  
> 
> 分析过程：
> - 打印简单的pdf(不超过一页内容)的使用原生的window.print()或者`vue-print-nb`，但是我们公司的pdf内容有十页左右，在实际测试中，不同电脑分页情况还不一样，效果不好。  
> - 查了下原因，因为每台电脑的dpi不同，导致打印的宽高度不一样。
> - 尝试用`html2canvas`，但是问题挺多的，超过一屏幕的内容就截取不了。 
> - 之前了解过`Puppeteer`，可以用来 生成页面pdf，还可以爬虫等等。 
> - `Puppeteer`默认会安装一个特定版本的`chrome`，因为在`node`接口是放在服务器上，`node`接口调用`Puppeteer`启动是`Puppeteer`安装的的chrome，那我只要适配这款`chrome`就行了，这样就不存在电脑差异问题了。  
>
说明：
1. 公司服务器是centos7
2. 前端vue
## 1. 安装Puppeteer 
`npm install  puppeteer --unsafe-perm=true`（要加`--unsafe-perm=true`否则会提示权限不足）
## 2. [安装Puppeteer依赖](https://github.com/puppeteer/puppeteer/blob/main/docs/troubleshooting.md#chrome-headless-doesnt-launch-on-unix)
先根据服务器安装的系统安装相关`Puppeteer`依赖，防止启动`Puppeteer`失败。
## 3. 生成pdf代码
```javascript
const puppeteer = require('puppeteer');
const account = 'xxxx';
const password = 'xxxxxx';

module.exports = {
  printPdf: async function (id,callBack) {
    const browser = await puppeteer.launch({ headless: true, args:['--no-sandbox']});//打开浏览器
    const page = await browser.newPage();//打开一个空白页
    await page.setViewport({width: 1920, height: 720});  // 设置视窗
    await page.goto('http://example.com/login',{ waitUntil: 'networkidle0' });
    // 输入账号密码
    await page.type(".ant-form-item:nth-child(1) .ant-input", account);
    await page.type(".ant-form-item:nth-child(2) .ant-input", password);
    // 登录
    await page.click('.login-form-button'),
    await page.waitForNavigation({ waitUntil: 'networkidle0' }),
    // 跳到报告页面
    await page.goto(`http://example.com/report/${id}`,{ waitUntil: 'networkidle0' });
    await page.pdf({
      path: 'res.pdf',
      printBackground:true,
      format:'A4',
      margin:{
       left:'5px',
       right:'5px',
      }
    });
    //关掉浏览器
    console.log('完成');
    await browser.close()
  }
}

```
## 4. node接口
```javascript
const http= require('http');
const url = require("url");
const querystring = require("querystring");
const fs= require('fs');

const printFn = require('./print-pdf.js')

const secret = require('./secret')  // 使用crypto-js解密id 这步骤可以省略
const server= http.createServer(function(request, response,next){
  // 设置跨域
  response.setHeader("Access-Control-Allow-Origin","*");
  //允许的header类型
  response.setHeader("Access-Control-Allow-Headers","*");
  //跨域允许的请求方式
  response.setHeader("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");
  if(request.method === "OPTIONS"){return response.end();}

  // 获取参数
  const requestUrl = request.url;
  const arg = url.parse(request.url).query;
  const params = querystring.parse(arg);

  if(requestUrl.indexOf( '/printPdf')>=0){
    let id = secret.Decrypt(params.id)  // 解密id
    printFn.printPdf(id).then(r=>{
      fs.readFile('./res.pdf', function(err, data){
        if(!err){
          // encode name 因为filename不能中文
          let realName = encodeURI(params.name,"GBK")
          realName = realName.toString('iso8859-1')
          // 返回pdf文件
          response.setHeader("Content-Type","application/pdf"); 
          response.setHeader("Content-Disposition","attachment; filename="+realName+'.pdf' )
          response.end(data);
        }else{
           response.end(JSON.stringify(err));
        }
      });
    })
  }
})
server.listen(8004)
```
> 当时遇到一个问题，前端接口发送到`node`了，但是没有返回，查看控制台接口状态是`cancel`，没有其他报错。
> 最后发现前端`axios`的`timeout`是5秒，但是我写的这个`node`接口大概7秒。修改`timeout`就可以了。
## 总结
这次接触`Puppeteer`,安装使用的时候各种报错，好在`github`都有这种`issue`，能够找到解决方案。这次也是第一次接触`node`，用`node`写接口,遇到跨域问题，返回请求头的问题等等。

---
### 20210823更新 

>> 最近又有打印需求，然后用express写了下。对比原生的`node`代码会更简洁，
> 顺便封装了下打印方法
```javascript
// main.js
const puppeteer = require('puppeteer');
const path = require('path')
const fs = require('fs'); 


const index = {
  resolvePath (p){
    return path.join(__dirname,p)
  },
  // 删除文件
  deleteFile(filepath){
    fs.unlinkSync(filepath);
  },
  // 打印
  async printFn({name,size,url}){
    const browser = await puppeteer.launch({ headless: true, args:['--no-sandbox']});//打开浏览器
    const page = await browser.newPage();//打开一个空白页
  
    await page.setViewport({width: 1920, height: 720});  // 设置视窗
    await page.goto(url,{ waitUntil: 'networkidle0' });
    await page.pdf({
      path: `./${name}.pdf`,
      printBackground:true,
      format: size || 'A4',
      margin:{
       left:'5px',
       right:'5px',
      }
    });
    //关掉浏览器
    console.log('完成');
    await browser.close();
  } 
}
module.exports = index
```
```javascript
// utils.js
const {printFn,resolvePath, deleteFile} = require('./utils.js')
const express = require('express')
var cors=require("cors");

const app = express()
const port = 3000
// 设置跨域
app.use(cors())

app.get('/index', async (req, res) => {
  const { name } = req.query
  const query =  { name, url: "https://www.baidu.com" }
  await printFn(query)
  try {
    const pdfPath = resolvePath(`../${name}.pdf`)
    res.download(pdfPath,()=>{
      // 下载后删除临时文件
      deleteFile(pdfPath)
    });
  } catch (error) {
    console.log(error)
  }
})

app.listen(port, () => {
  console.log(`node app listening at http://localhost:${port}`)
})
```

