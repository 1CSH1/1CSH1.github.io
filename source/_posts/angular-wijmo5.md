---
title: 在 Angular 项目中配置 Wijmo5
date: 2017-06-18 22:00:00
categories: [Angular, Wijmo5]
tags: [Angular, Wijmo5]
---

> 摘要：本文将介绍在 Angular4 项目中配置 Wijmo5 UI控件，文中采用的 Angular 版本为 4.0.0，Wijmo 版本为 C1Wijmo-Eval_5.20171.300


### 下载 Wijmo5 官方文件

[C1Wijmo-Eval_5.20171.300.zip](https://1csh1.github.io/file/angular-wijmo5/C1Wijmo-Eval_5.20171.300.zip)

### 创建好 Angular4 项目

文中不介绍搭建 Angular 开发环境的过程，需要的可以参考官方提供的 [快速入门](https://angular.io/guide/quickstart)

创建项目 **ng new wijmo** ，并测试 **ng serve** ，运行成功过后再进行接下来的配置。

### 配置 Wijmo5

解压文件后进入 NpmImages 文件夹，拷贝 wijmo-amd-min 到 Angular项目 wijmo 的 npm_modules 目录下且改名为 wijmo，并 wijmo 项目的 package.json 文件中的 dependencies 中添加配置如下 

```
"dependencies": {
    "wijmo": "^5.20171.300"
}
```

将 “C1Wijmo-Eval_5.20171.300\Dist\styles” 拷贝到 Angular 项目的 src 目录下，并在 src/styles.css 文件中添加 wijmo 自带的样式文件

```
@import "styles/wijmo.min.css";
```

### 编写代码

 **app/app.modules.ts** 代码如下：

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { WjInputModule } from 'wijmo/wijmo.angular2.input';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    WjInputModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

添加了 ``` import { WjInputModule } from 'wijmo/wijmo.angular2.input'; ``` 和 ```  imports: [ WjInputModule ] ```


**app/component.ts** 代码如下：

```
import { Component } from '@angular/core';
import { WjInputModule } from 'wijmo/wijmo.angular2.input';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app';

  value: number = 123;

  change() {
    alert("my WjInputNumber value is " + this.value);
  }
}
```

添加了

```
import { WjInputModule } from 'wijmo/wijmo.angular2.input';

  value: number = 123;
  change() {
    alert("my WjInputNumber value is " + this.value);
  }

```

**app/app.component.html**添加代码如下：

```
<h1>My First Wijmo Project</h1>
<wj-input-number
  [(value)]="value"
  (change)="change()"
></wj-input-number>
```

### 运行结果

![运行结果](https://1csh1.github.io/img/angular-wijmo5/angular-wijmo5-result.png)
 
