---
title: Angular4 中显示内容的 CSS 样式
date: 2017-11-05 21:00:00
categories: [Angular]
tags: [Angular]
---


> 摘要：本文简单介绍在使用 Angular4 的时候，显示要内容的 CSS 样式

### 问题
Angular 中有 innerHTML 属性来设置要显示的内容，但是如果内容包含 CSS 样式，无法显示样式的效果。
比如：


```
public content: string = "<div style='font-size:30px'>Hello Angular</div>";

<p [innerHTML]="content"></p>
```

只会显示 Hello World ,字体不会是 30px，也就是 CSS 样式没有效果。

### 解决方案
自定义一个 Pipe 来对内容做转换。看下面代码。
#### 写一个 HtmlPipe 类

```
import {Pipe, PipeTransform} from "@angular/core";
import {DomSanitizer} from "@angular/platform-browser";

@Pipe({
  name: "html"
})

export class HtmlPipe implements PipeTransform{

  constructor (private sanitizer: DomSanitizer) {

  }

  transform(style) {
    return this.sanitizer.bypassSecurityTrustHtml(style);
  }
}

```

#### 在需要的模块里面引入管道 HtmlPipe

```
@NgModule({
  declarations: [
    HtmlPipe
  ]
})
```

#### 在 innerHtml 中增加管道名字

```
<p [innerHTML]="content | html"></p>
```

这样就可以显示 content 的 CSS 样式。

