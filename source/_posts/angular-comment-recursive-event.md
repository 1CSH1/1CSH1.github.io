---
title: Angular 实现类似博客获取回复评论的数据
date: 2017-08-01 22:30:00
categories: [Angular, James-Blog]
tags: [Angular, 评论, 递归]
---

> 摘要：本文继上一篇文章 [Angular 实现类似博客评论的递归显示](https://1csh1.github.io/2017/07/31/angular-comment-recursive-display/) 讲述的内容，上篇文章里面只是说明了如何实现评论梯形显示，在博客评论中我们经常看到可以回复某一条评论，本文讲述如何实现点击某一条评论的回复按钮后，获取该条评论的内容并显示在输入框中。类似 CSDN 博客评论一样，点击回复后输入框自动添加了 [reply]u011642663[/reply]

### 思路
依据上一篇文章中的评论梯形显示，我们还需要实现点击回复后，屏幕自动到达输入框位置，并且获取了点击回复的评论的信息。首先分解一下这个功能点，在项目中我们也会经常分解功能点，这个功能点有 2 个小点：一是在每条评论中加上 [回复] 按钮，点击回复后跳到输入框位置；二是点击回复后，获取到点击回复的那条评论的信息。下面我们一一解决。

### 跳转到输入框
我们接触前段第一个语言便是 HTML，我们知道 HTML 中有一个 # 定位，下面代码简单解释一下。
假设这个 HTML 代码文件是 index.html
```
<html>
  <head>
  </head>
  <body>
    <a href="index.html#pointer">Click me to pointer</a>
    <div id="pointer">
      <h1>哈哈哈哈</h1>
    </div>
  </body>
</html>
```

上面代码只要点击 Click me to pointer 这个链接，页面就会跳到 id="pointer" 这个 div 的位置。所以我们在实现这个点击回复跳转到输入框中就可以使用这个方法。
我们在 comment-component.html 中将评论输入框加入 id="comment"，接下来就是路径拼接的问题了，我们可以通过 Angular 的 Router 的 url 来获取本页面的路径，然后在这个路径后面加入 #comment 就可以实现跳转了，下面是实现这个跳转功能的代码

#### 添加 id="comment"
comment-component.html   
```
<!-- Comment -->
<div class="container font-small">
  <div class="row">
    <div class="col-lg-8 offset-lg-2 col-md-10 offset-md-1">

      <comment-view [comments]="comments" (contentEvent)="getReplyComment($event)" ></comment-view>

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

#### 添加通过路由获取当前页面 URL
comment-view.component.ts  
```
@Component({
  selector: 'comment-view',
  templateUrl: './comment-view.component.html',
  styleUrls: ['./comment-view.component.css']
})
export class CommentViewComponent implements OnInit {
  @Input()
  public comments: Comment[];

  // 用于跳转到回复输入框的url拼接
  public url: string = "";

  constructor(private router: Router,
              private activateRoute: ActivatedRoute ) {
  }

  ngOnInit(): void {
    this.url = this.router.url;
    this.url = this.url.split("#")[0];
    this.url = this.url + "#comment";
  }
}

```

#### 添加链接 href="{{url}}"
comment-view.component.html   
```
<div *ngFor="let comment of comments">
  <div class="media">
    <div class="pull-left">
      <span class="media-object"></span>
    </div>
    <div class="media-body">
      <h4 class="media-heading">{{ comment.username }}
        <small class="pull-right">{{ comment.time }}&nbsp;|&nbsp;<a href="{{url}}" (click)="reply(comment)" >{{ 'comment.reply' | translate }}</a></small>
      </h4>
       {{ comment.content }}
      <hr>
      <comment-view *ngIf="comment.cComments != null" [comments]="comment.cComments" (contentEvent)="transferToParent($event)"></comment-view>
    </div>
  </div>
</div>

```

这就实现了页面跳转的功能点，接下来实现获取回复的评论的信息。

### 获取回复的评论信息
有人会说获取回复的评论信息，这不简单么？加个 click 事件不就行了。还记得上一篇文章咱们是如何实现梯形展示评论的么？咱们是通过递归来实现的，怎么添加 click 事件让一个不知道嵌了多少层的组件能够把评论信息传给父组件？首先不具体想怎么实现，我们这个思路是不是对的：把子组件的信息传给父组件？答案是肯定的，我们就是要把不管嵌了多少层的子组件的信息传给 comment.component.ts 这个评论模块的主组件。
Angular 提供了 @Output 来实现子组件向父组件传递信息，我们在 comment-view.component.ts 模块中添加 @Output 向每个调用它的父组件传信息，我们是嵌套的，这样一层一层传出来，直到传给 comment-component.ts 组件。我们看代码怎么实现。

#### 实现代码

comment-view.component.ts

```
@Component({
  selector: 'comment-view',
  templateUrl: './comment-view.component.html',
  styleUrls: ['./comment-view.component.css']
})
export class CommentViewComponent implements OnInit {
  @Input()
  public comments: Comment[];
  // 点击回复时返回数据
  @Output()
  public contentEvent: EventEmitter<Comment> = new EventEmitter<Comment>();
  // 用于跳转到回复输入框的url拼接
  public url: string = "";

  constructor(private router: Router,
              private activateRoute: ActivatedRoute ) {
  }

  ngOnInit(): void {
    this.url = this.router.url;
    this.url = this.url.split("#")[0];
    this.url = this.url + "#comment";
  }

  reply(comment: Comment) {
    this.contentEvent.emit(comment);
  }

  transferToParent(event) {
    this.contentEvent.emit(event);
  }
}
```

comment-view.component.html

```
<div *ngFor="let comment of comments">
  <div class="media">
    <div class="pull-left">
      <span class="media-object"></span>
    </div>
    <div class="media-body">
      <h4 class="media-heading">{{ comment.username }}
        <small class="pull-right">{{ comment.time }}&nbsp;|&nbsp;<a href="{{url}}" (click)="reply(comment)" >{{ 'comment.reply' | translate }}</a></small>
      </h4>
       {{ comment.content }}
      <hr>
      <comment-view *ngIf="comment.cComments != null" [comments]="comment.cComments" (contentEvent)="transferToParent($event)"></comment-view>
    </div>
  </div>
</div>
```


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

  // 要回复的评论
  public replyComment: Comment = new Comment();

  public id: number = 0;
  public content: string = "";

  ngOnInit(): void {
  }

  getReplyComment(event) {
    this.replyComment = event;
    this.id = this.replyComment.id;
    this.content = "[reply]" + this.replyComment.username + "[reply]\n";
  }
}
```

comment.component.html
```
<!-- Comment -->
<div class="container font-small">
  <div class="row">
    <div class="col-lg-8 offset-lg-2 col-md-10 offset-md-1">

      <comment-view [comments]="comments" (contentEvent)="getReplyComment($event)" ></comment-view>

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

解释一下代码逻辑：
我们在 comment-view.component.ts 添加以下几点：
1. 定义了@Output() contentEvent
2. 添加了reply(comment: Comment) 事件在点击回复的时候触发的，触发的时候 contentEvent 将 comment 传到父模块
3. 添加 transferToParent(event) 是接受子组件传来的 event, 并且继续将 event 传到父组件

在 comment.component.ts 中定义了 getReplyComment(event) 方法，该方法接收子组件传递来的评论信息，并将信息显示在页面上。大功告成。。。

### 效果图
![angular-comment-recursive-event.png](https://1csh1.github.io/img/angular-comment-recursive-event/angular-comment-recursive-event.png)

### demo 代码
[James-Blog](https://github.com/1CSH1/james-blog-ui/tree/e025e62e17461a6937e9b6d3e0865e1ae7c2e4d4)


