---
title: 在 Angular 项目中添加 clean-blog 模板
date: 2017-07-03 23:00:00
categories: [Angular, James-Blog]
tags: [Angular, clean-blog]
---

> 摘要：本文将介绍在 Angular4 项目中添加网上开源 Bootstrap 博客模板 ** clean-blog **

### clean-blog 博客模板下载
[clean-blog](https://startbootstrap.com/template-overviews/clean-blog/)
或者在下面链接下载
[startbootstrap-clean-blog-4-dev.zip](https://1csh1.github.io/file/angular-add-clean-blog-template/startbootstrap-clean-blog-4-dev.zip)

### 解压并拷贝
解压下载的文件,将所有文件拷贝到 ** assets/clean-blog ** 目录下
![angular-add-clean-blog-template-1.png](https://1csh1.github.io/img/angular-add-clean-blog-template/angular-add-clean-blog-template-1.png)

### 拷贝代码

#### 将 clean-blog 的 index.html 内容拷贝到 app.component.html

```
<!--The whole content below can be removed with the new code.-->

<!-- Navigation -->
<nav class="navbar fixed-top navbar-toggleable-md navbar-light" id="mainNav">
  <div class="container">
    <button class="navbar-toggler navbar-toggler-right" type="button" data-toggle="collapse"
            data-target="#navbarResponsive" aria-controls="navbarResponsive" aria-expanded="false"
            aria-label="Toggle navigation">
      Menu <i class="fa fa-bars"></i>
    </button>
    <a class="navbar-brand" href="index.html">Start Bootstrap</a>
    <div class="collapse navbar-collapse" id="navbarResponsive">
      <ul class="navbar-nav ml-auto">
        <li class="nav-item">
          <a class="nav-link" href="index.html">Home</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="../assets/clean-blog/about.html">About</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="../assets/clean-blog/post.html">Sample Post</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="../assets/clean-blog/contact.html">Contact</a>
        </li>
      </ul>
    </div>
  </div>
</nav>

<!-- Page Header -->
<header class="masthead" style="background-image: url('../assets/clean-blog/img/home-bg.jpg')">
  <div class="container">
    <div class="row">
      <div class="col-lg-8 offset-lg-2 col-md-10 offset-md-1">
        <div class="site-heading">
          <h1>Clean Blog</h1>
          <span class="subheading">A Blog Theme by Start Bootstrap</span>
        </div>
      </div>
    </div>
  </div>
</header>

<!-- Main Content -->
<div class="container">
  <div class="row">
    <div class="col-lg-8 offset-lg-2 col-md-10 offset-md-1">
      <div class="post-preview">
        <a href="../assets/clean-blog/post.html">
          <h2 class="post-title">
            Man must explore, and this is exploration at its greatest
          </h2>
          <h3 class="post-subtitle">
            Problems look mighty small from 150 miles up
          </h3>
        </a>
        <p class="post-meta">Posted by <a href="#">Start Bootstrap</a> on September 24, 2017</p>
      </div>
      <hr>
      <div class="post-preview">
        <a href="../assets/clean-blog/post.html">
          <h2 class="post-title">
            I believe every human has a finite number of heartbeats. I don't intend to waste any of mine.
          </h2>
        </a>
        <p class="post-meta">Posted by <a href="#">Start Bootstrap</a> on September 18, 2017</p>
      </div>
      <hr>
      <div class="post-preview">
        <a href="../assets/clean-blog/post.html">
          <h2 class="post-title">
            Science has not yet mastered prophecy
          </h2>
          <h3 class="post-subtitle">
            We predict too much for the next year and yet far too little for the next ten.
          </h3>
        </a>
        <p class="post-meta">Posted by <a href="#">Start Bootstrap</a> on August 24, 2017</p>
      </div>
      <hr>
      <div class="post-preview">
        <a href="../assets/clean-blog/post.html">
          <h2 class="post-title">
            Failure is not an option
          </h2>
          <h3 class="post-subtitle">
            Many say exploration is part of our destiny, but it’s actually our duty to future generations.
          </h3>
        </a>
        <p class="post-meta">Posted by <a href="#">Start Bootstrap</a> on July 8, 2017</p>
      </div>
      <hr>
      <!-- Pager -->
      <div class="clearfix">
        <a class="btn btn-secondary float-right" href="#">Older Posts &rarr;</a>
      </div>
    </div>
  </div>
</div>

<hr>

<!-- Footer -->
<footer>
  <div class="container">
    <div class="row">
      <div class="col-lg-8 offset-lg-2 col-md-10 offset-md-1">
        <ul class="list-inline text-center">
          <li class="list-inline-item">
            <a href="#">
              <span class="fa-stack fa-lg">
                  <i class="fa fa-circle fa-stack-2x"></i>
                  <i class="fa fa-twitter fa-stack-1x fa-inverse"></i>
              </span>
            </a>
          </li>
          <li class="list-inline-item">
            <a href="#">
              <span class="fa-stack fa-lg">
                  <i class="fa fa-circle fa-stack-2x"></i>
                  <i class="fa fa-facebook fa-stack-1x fa-inverse"></i>
              </span>
            </a>
          </li>
          <li class="list-inline-item">
            <a href="#">
              <span class="fa-stack fa-lg">
                  <i class="fa fa-circle fa-stack-2x"></i>
                  <i class="fa fa-github fa-stack-1x fa-inverse"></i>
              </span>
            </a>
          </li>
        </ul>
        <p class="copyright text-muted">Copyright &copy; Your Website 2017</p>
      </div>
    </div>
  </div>
</footer>

```

#### ** styles.css ** 添加代码

```
/* clean-blog */
@import "assets/clean-blog/vendor/font-awesome/css/font-awesome.min.css";
@import "https://fonts.googleapis.com/css?family=Lora:400,700,400italic,700italic";
@import "https://fonts.googleapis.com/css?family=Open+Sans:300italic,400italic,600italic,700italic,800italic,400,300,600,700,800";
@import "assets/clean-blog/css/clean-blog.min.css";

.navbar-toggler {
  z-index: 1;
}

@media (max-width: 576px) {
  nav > .container {
    width: 100%;
  }
}
```

#### ** index.html ** 添加 JS 引用
```
<!-- clean-blog -->
<script src="assets/clean-blog/js/clean-blog.min.js"></script>
```

### 结果

![angular-add-clean-blog-template-2.png](https://1csh1.github.io/img/angular-add-clean-blog-template/angular-add-clean-blog-template-2.png)

** 提示: **  文中的配置只是把 clean-blog 给搭建在 Angular4 的项目中，但是并没有大改其中的东西，比如链接等，现在还是一个静态的网页，后期才会添加自己的代码上去

### 示例代码
[angular + clean-blog](https://github.com/1CSH1/james-blog-ui/tree/4505a4fa26e2835b33914be1976b5aaac1254e20)


