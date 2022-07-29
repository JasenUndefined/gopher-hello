## Nest 简单实现自定义装饰器

> 携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第1天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

## 装饰器

ES6 中装饰器是一个表达式，一种函数，是对类或者类方法进行处理的函数。在很多面向对象语言都有此功能。主要作用就是增加或修改类的功能。

```js
@ + 函数名

// 例子-- 类装饰器
@AddLogger()
class UserClass {
    // 例子 -- 方法装饰器
    @setPropty()
    doHandler(){

    }
}
```

以上的例子就是类装饰器和方法装饰器。装饰器是 AOP 模式的一种实现方式，所以它的作用主要是用于增加或修改类的功能，其是在编译时发生的，而不是在运行时。

## Nest 中的参数装饰器

参数装饰器是 Nest 框架中提供一组实用方法，其可以结合 Http 路由处理器一起使用。

- @Request

- @Response

- @Query

- @Body

以上只列举，其还有其他的参数装饰器。使用方式如下：

```ts
class UserController {
    getUserInfo(@Request() req, @Query('id') id){
      return  this.service.getUserbyId(id)
    }
}
```

@Query 相当于 req.query, @Query('id') 方式就可以获取到 query 中的参数。

## Nest 中自定义装饰器

装饰器其实就是一种函数，所以我们可以先定义一个我们需要的功能函数，比如设置用户属性 setUser，每次请求时都计数加1，统计用户请求计数。

```ts
const setUser = function(){
    const user = new User()
    user.requestCount ++;
}
```

单纯定义一个函数，还需要将该函数定义为是一个装饰器，且函数中每次计数是新 new 一个用户，所以还应该是能够接受参数。所以要让一种函数成为一个装饰器，方式如下：

1、createParamDecorator

```ts
export const setUser = createParamDecorator((data:string,ctx ExecutionContext)=>{
      const request = ctx.switchToHttp().getRequest();
      const user = request.user;
      user.requestCount ++;
      return data? user && user[data] : user;
})


// 使用
export class UserClass{
    getUserInfo(@setUser('id') id string){
     // do something
    }
}
```

2、使用管道

```ts
@Get()
async findOne(@User(new ValidationPipe()) user: UserEntity) {
  console.log(user);
}
```

3、装饰器聚合 applyDecorators

```ts
export const doSet = ()=>{
    const req = Context.getRequest();
    const user = req.user
    user.requestCount ++; 
}

export const setUser =()=>{
    return applyDecorators(
      SetMetadata('user', user),
      doSet()
    )
}

// 使用
export class UserClass{

    @setUser()
    getUserInfo(@setUser('id') id string){
     // do something
    }
}
```

将自定义装饰器 setUser 装饰到方法上，这样每次调用该方法都会触发 setUser 的方法。

参考资料：

- https://docs.nestjs.cn/8/customdecorators
