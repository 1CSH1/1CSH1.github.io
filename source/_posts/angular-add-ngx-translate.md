---
title: 在 Angular 项目中添加 i18n 插件 ngx-translate
date: 2017-07-01 18:00:00
categories: [Angular, ngx-translate]
tags: [Angular, ngx-translate, i18n]
---

> 摘要：本文将介绍在 Angular4 项目中配置 ngx-translate i18n 国际化组件

### npm 安装 ngx-translate 模块

```
npm install @ngx-translate/core --save
npm install @ngx-translate/http-loader --save
```

### 在 Angular 项目配置

#### app.module.ts

添加
```
import { TranslateLoader, TranslateModule} from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';

  imports: [
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useFactory: (createTranslateHttpLoader),
        deps: [Http]
      }
    })
  ]

```

结果如下： 

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpModule, Http } from '@angular/http';
import { TranslateLoader, TranslateModule} from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';

import { AppComponent } from './app.component';

export function createTranslateHttpLoader(http: Http) {
  return new TranslateHttpLoader(http, './assets/i18n/', '.json');
}

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HttpModule,
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useFactory: (createTranslateHttpLoader),
        deps: [Http]
      }
    })
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

#### app.component.ts

添加

```
import { TranslateService } from "@ngx-translate/core";

  constructor(public translateService: TranslateService) {

  }
  
  ngOnInit() {
    // --- set i18n begin ---
    this.translateService.addLangs(["zh", "en"]);
    this.translateService.setDefaultLang("zh");
    const browserLang = this.translateService.getBrowserLang();
    this.translateService.use(browserLang.match(/zh|en/) ? browserLang : 'zh');
    // --- set i18n end ---
  }
```

结果如下：

```
import { Component } from '@angular/core';
import { TranslateService } from "@ngx-translate/core";

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app';

  constructor(public translateService: TranslateService) {

  }

  ngOnInit() {
    // --- set i18n begin ---
    this.translateService.addLangs(["zh", "en"]);
    this.translateService.setDefaultLang("zh");
    const browserLang = this.translateService.getBrowserLang();
    this.translateService.use(browserLang.match(/zh|en/) ? browserLang : 'zh');
    // --- set i18n end ---
  }
}

```

#### 添加多语言文件

在 ** src/app/assets/ ** 下创建 ** i18n ** 文件夹，并在文件夹内创建 ** en.json ** 和 ** zh.json ** 文件
![angular-i18n-folder](https://1csh1.github.io/img/angular-add-ngx-translate/angular-i18n-folder.png)

### 测试

#### app.component.html

添加代码：
```
<div>
  <span> test the i18n module: ngx-translate</span>
  <h1>{{ 'hello' | translate }}</h1>
</div>
```

#### 在 en.json 和 zh.json 文件中添加配置

** en.json **
```
{
  "hello": "the word is hello"
}
```
** zh.json **
```
{
  "hello": "你好"
}
```

### 测试结果
在中文下
![angular-i18n-result-zh](https://1csh1.github.io/img/angular-add-ngx-translate/angular-i18n-result-zh.png)

在英文下
![angular-i18n-result-en](https://1csh1.github.io/img/angular-add-ngx-translate/angular-i18n-result-en.png)

### 示例代码
[angular + ngx-translate](https://github.com/1CSH1/james-blog-ui/tree/7d2a64c3bc97d33e1b696019536f7627bcc3f3c4)

### 参考文章
[ngx-translate core](https://github.com/ngx-translate/core)
