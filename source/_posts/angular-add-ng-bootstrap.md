---
title: 在 Angular 项目中添加插件 ng-bootstrap
date: 2017-07-02 20:00:00
categories: [Angular, ng-bootstrap]
tags: [Angular, ng-bootstrap]
---

> 摘要：本文将介绍在 Angular4 项目中配置 ng-bootstrap

### npm 安装 ng-bootstrap 模块

```
npm install @ng-bootstrap/ng-bootstrap --save
```

### 在 Angular 项目配置

#### app.module.ts

添加
```
import { NgbModule } from "@ng-bootstrap/ng-bootstrap";

  imports: [
    /**
     * ngx-bootstrap
     */
    NgbModule.forRoot()
  ],

```

#### 添加 bootstrap.min.css 样式
在 ** assets ** 文件夹下 ** bootstrap/bootstrap.min.css **， 并在 ** style.css ** 文件中添加

```
@import "assets/bootstrap/bootstrap.min.css";
```

### 测试

#### app.component.html

添加代码：
```
<div>
  <span> test the ng-bootstrap</span>
  <div [(ngModel)]="model" ngbRadioGroup name="radioBasic">
    <label class="btn btn-primary">
      <input type="radio" [value]="1"> Left (pre-checked)
    </label>
    <label class="btn btn-primary">
      <input type="radio" value="middle"> Middle
    </label>
    <label class="btn btn-primary">
      <input type="radio" [value]="false"> Right
    </label>
  </div>
  <hr>
  <pre>{{model}}</pre>
</div>
```


### 测试结果

![angular-ng-bootstrap-result](https://1csh1.github.io/img/angular-add-ng-bootstrap/angular-ng-bootstrap-result.png)

### 示例代码
[angular + ng-bootstrap](https://github.com/1CSH1/james-blog-ui/tree/6c2c1503b4102c28b14f8b9e7fb447f8d52e1548)

### 参考文章
[NG Bootstrap - Angular directives specific to Bootstrap 4](https://github.com/ng-bootstrap/ng-bootstrap)
[Bootstrap 4 components, powered by Angular](https://ng-bootstrap.github.io/#/home)
[ngx-translate core](https://github.com/ngx-translate/core)
