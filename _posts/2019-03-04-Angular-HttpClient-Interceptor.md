---
layout: post
title: "Angular HttpClient中的拦截器 "
date: 2019-03-04
author: "WangX"
catalog: true
header-style: text
tags:
    - Angular
    - HttpClient
---

## 概念
* HTTP 拦截机制是 @angular/common/http 中的主要特性之一。可以通过拦截器来监视和转换从应用发送到服务器的HTTP&**请求**。同时也可以监视和转换从服务器返回的**响应**。
* 拦截器可以实现对每次HTTP请求/响应的隐式操作，若没有拦截器只能显示实现这些操作。
* 要实现拦截器，就要实现一个实现了 HttpInterceptor 接口中的 intercept() 方法的类。intercept 方法会把**请求转换成一个最终返回 HTTP 响应体的 Observable**。
* 大多数拦截器拦截都会在传入时检查请求，然后把（可能被修改过的）请求转发给 next 对象的 handle() 方法，而 next 对象实现了 HttpHandler 接口。
* 像 intercept() 一样，handle() 方法也会把 HTTP 请求转换成 HttpEvents 组成的 Observable，它最终包含的是来自服务器的响应。 intercept() 函数可以检查这个可观察对象，并在把它返回给调用者之前修改它。

```typescript
export class NoopInterceptor implements HttpInterceptor {

  intercept(req: HttpRequest<any>, next: HttpHandler):
    Observable<HttpEvent<any>> {
    return next.handle(req);
  }
}
```
上述代码中实现的拦截器，没有对请求进行任何额外的操作，直接将请求转发给了next对象的handle方法。

* next 对象表示拦截器链表中的下一个拦截器。 这个链表中的最后一个 next 对象就是 HttpClient 的后端处理器（backend handler），它会把请求发给服务器，并接收服务器的响应。
* 大多数的拦截器都会调用 next.handle()，以便这个请求流能走到下一个拦截器，并最终传给后端处理器。 
* 拦截器也可以不调用 next.handle()，使这个链路短路，并返回一个带有人工构造出来的服务器响应的 自己的 Observable。这是一种常见的中间件模式。
* 拦截器也是一种服务，必须先provide这个拦截器，才能使用。
* 由于拦截器是 HttpClient 服务的（可选）依赖，所以**必须在提供 HttpClient 的同一个（或其各级父注入器）注入器中提供这些拦截器。 那些在 DI 创建完 HttpClient 之后再提供的拦截器将会被忽略**。
* Angular 会按照你提供它们的顺序应用这些拦截器。 如果你提供拦截器的顺序是先 A，再 B，再 C，那么请求阶段的执行顺序就是 A->B->C，而响应阶段的执行顺序则是 C->B->A。
*  intercept() 和 handle() 方法返回的是 HttpEvent<any> 的可观察对象。这是因为拦截器工作的层级比那些 HttpClient 方法更低一些。每个 HTTP 请求都可能会生成很多个事件，包括上传和下载的进度事件。 实际上，HttpResponse 类本身就是一个事件，它的类型（type）是 HttpEventType.HttpResponseEvent。
* 拦截器有能力改变请求和响应，但 HttpRequest 和 HttpResponse 实例的属性却是只读（readonly）的， 因此让它们基本上是不可变的。要想修改该请求，就要先克隆它，并修改这个克隆体，然后再把这个克隆体传给 next.handle()。
* 把克隆后的请求体设置成 null， Angular则会清空这个请求体了。果你把克隆后的请求体设置成undefined，Angular则会让这个请求体保持原样。

## 例子
* 在每个http请求的头中，加入token

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {

    constructor(private router: Router) {}

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        const token = localStorage.getItem('token');
        if (token && this.router.url !== '/login') {
            const authReq = req.clone({ setHeaders: { 'token': token } });
            return next.handle(authReq);
        } else {
            return next.handle(req);
        }
    }
}

```
* 对每个http url 进行处理

```typescript
@Injectable()
export class DecorateUrlInterceptor implements HttpInterceptor {

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

        const decorateReq = req.clone({
            url: 'https://localhost:18443' + req.url
        });

        return next.handle(decorateReq);
    }
}
```

* 对每个http请求的返回内容，进行处理，判断token是否过期

```typescript
@Injectable()
export class TokenExpiredInterceptor implements HttpInterceptor {

    constructor(private router: Router, private loginService: LoginService) {}

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        return next.handle(req)
            .pipe(
                tap(
                    undefined,
                    err => {
                        if (err instanceof HttpErrorResponse && err.status === 401) {
                            alter(err.error.message);
                        }
                    }
                )
            );
    }
}
```