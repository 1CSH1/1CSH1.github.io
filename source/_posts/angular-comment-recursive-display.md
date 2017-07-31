---
title: Angular 实现类似博客评论的递归显示
date: 2017-07-31 22:30:00
categories: [Angular, James-Blog]
tags: [Angular, 评论, 递归]
---

> 摘要：本文讲述用 Angular4 来实现一般博客中经常看到的递归评论模块


我们在一些技术博客中会经常看到很多递归评论，也即是我们可以回复博友的评论，且界面很美观，有梯度的显示格式，日前在空余时间写类似的 demo，所以记录下来，可以给需要的人一些借鉴的作用。
好了，废话少说，直奔主题。。。

### 思路
我们在写后台程序的时候，经常会遇到生成类似树的这种数据结构，我们直觉就是使用递归的方法来实现，起初我也是这么想的，就是写一个 Angular4 的递归方法，组成一个字符串，然后在界面显示，类似下面代码

```
@Component({
  selector: "comment",
  template: '{{ comments }}'
})
export class CommentComponent {

  public comments: string = "";

  generateComment(Comment comment) {
    this.comments = this.comments + "<div>" + comment.content + "</div>";
    if (comment.pComment != null) {
      generateComment(comment.pComment);
    }
  }
}
```

很天真的以为可以了，结果一试，标签不会被解析，才想起已经过了解析标签的过程了。。。


后来想着，现在的前端框架都是以组件化自称， Angular4 也不例外，那么一个 Component 可以嵌入任何 Component，那肯定可以嵌入自己，也就有了类似递归的概念了，想到这立马试试。。。

### 具体实现

思路是这样子，我定义了数据的格式，是每个评论下面有一个子评论数组，而不是每个评论有一个父评论，数据格式如下：

```
"comments": 
     [
        {
          "id": 1,
          "username": "James1",
          "time": "2017-07-09 21:02:21",
          "content": "哈哈哈1<h1>哈哈哈</h1>",
          "status": 1,
          "email": "1xxxx@xx.com",
          "cComments": [
              {
              "id": 2,
              "username": "James2",
              "time": "2017-07-09 21:02:22",
              "content": "哈哈哈2",
              "status": 1,
              "email": "2xxxx@xx.com",
              "cComments": null
            }
          ]
        }
     ]
```

CommentComponent 组件实现了评论模块，但是递归评论并不在这个组件实现，而是在子组件 CommentViewComponent 实现，因为 CommentComponent 还包括一个一个输入评论的文本框。 

评论总模块 ComponentComponent 代码：

comment.component.ts
```
@Component({
  selector: 'comment',
  templateUrl: './comment.component.html',
  styleUrls: ['./comment.component.css']
})
export class CommentComponent implements OnInit {

  @Input()
  public comments: Comment[];

  ngOnInit(): void {
  }
}
```

comment.component.html
```
<div class="container font-small">
  <div class="row">
    <div class="col-lg-8 offset-lg-2 col-md-10 offset-md-1">

      <comment-view [comments]="comments"></comment-view>

      <div class="well" id="comment">
        <h4>{{ 'comment.leaveComment' | translate }}</h4>
        <form role="form">
          <div class="form-group">
            <input type="hidden" [(ngModel)]="id" name="id">
            <textarea [(ngModel)]="content" name="content" class="form-control" rows="5"></textarea>
          </div>
          <button type="submit" class="btn btn-primary">Submit</button>
        </form>
      </div>
    </div>
  </div>
</div>
```

comment.component.css
```
.media {
  font-size: 14px;
}

.media-object {
  padding-left: 10px;
}
```

子模块 ComponentViewComponent 代码：

component-view.component.ts
```
@Component({
  selector: 'comment-view',
  templateUrl: './comment-view.component.html',
  styleUrls: ['./comment-view.component.css']
})
export class CommentViewComponent implements OnInit {
  @Input()
  public comments: Comment[];

  constructor(private router: Router,
              private activateRoute: ActivatedRoute ) {
  }

  ngOnInit(): void {
  }
}

```

component-view.component.html
```
<div *ngFor="let comment of comments">
  <div class="media">
    <div class="pull-left">
      <span class="media-object"></span>
    </div>
    <div class="media-body">
      <h4 class="media-heading">{{ comment.username }}
        <small class="pull-right">{{ comment.time }}&nbsp;|&nbsp;<a href="#" >{{ 'comment.reply' | translate }}</a></small>
      </h4>
       {{ comment.content }}
      <hr>
      <comment-view *ngIf="comment.cComments != null" [comments]="comment.cComments"></comment-view>
    </div>
  </div>
</div>

```

comonent-view.component.css
```
.media {
  font-size: 14px;
}

.media-object {
  padding-left: 10px;
}
```

### 结果
这时的展示结果如下图所示：


![angular-comment-recursive-display.png](https://1csh1.github.io/img/angular-comment-recursive-display/angular-comment-recursive-display.png)



