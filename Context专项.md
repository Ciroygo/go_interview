# 是什么？
`context`是Go语言提供的一个标准库，用于在多个goroutine之间传递请求作用域、取消信号和请求间的元数据。通过在goroutine之间传递上下文对象，可以使得goroutine能够正确的控制请求的生命周期，同时也可以避免使用全局变量和同步原语等方式来共享数据。

`context`主要用于以下几个方面：

1.  控制请求的超时和取消：通过在上下文中设置定时器，可以在一定时间后自动取消请求，防止请求无限阻塞；同时，也可以在主函数中发送取消信号，通知各个goroutine结束当前操作，避免因为某一个操作的错误导致整个请求无法正常结束。
    
2.  传递请求作用域和元数据：通过在上下文中设置键值对，可以传递请求相关的元数据，如请求ID、用户认证信息等；同时，也可以使用上下文对象在多个goroutine之间共享请求作用域，比如在不同的层级中获取同一个请求的日志、追踪和统计信息。
    
3.  控制goroutine的生命周期：通过在上下文中设置一个cancel函数，可以在某些情况下强制终止goroutine的执行，如在服务关闭时清理所有goroutine。
    

在实际应用中，可以在HTTP请求、gRPC调用、数据库访问等各种场景中使用上下文对象来管理请求的生命周期、控制并发和共享请求相关的信息。

# 控制请求的超时和取消

可以使用`context`来控制请求的超时和取消，具体实现步骤如下：

1.  创建一个带有超时时间的`context`对象：使用`context.WithTimeout`函数可以创建一个带有超时时间的`context`对象，这个超时时间表示在多长时间内必须完成操作，否则就会自动取消请求。

```go
ctx, cancel := context.WithTimeout(context.Background(), time.Second * 10)
defer cancel() // 最后一定要调用cancel，避免goroutine泄漏

```

2.  在需要进行超时控制的操作中使用`select`语句：在操作中使用`select`语句监听`context.Done()`通道和操作完成通道，如果`context.Done()`通道被关闭，则说明已经超时或者取消了请求，这时可以直接退出操作；如果操作完成通道被关闭，则说明操作已经完成，可以继续执行下一步操作。

```go
select {
case <-ctx.Done():
    // 请求超时或者取消，退出操作
    return ctx.Err()
case result := <-operation:
    // 操作完成，继续执行下一步操作
    // ...
}

```

3.  在主函数中发送取消信号：在主函数中可以调用`cancel`函数，通知`context`对象取消请求，这会导致在所有使用了该`context`对象的goroutine中调用`context.Done()`通道。

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    ctx, cancel := context.WithTimeout(context.Background(), time.Second * 10)
    defer cancel()

    // 执行耗时操作，如果超时或者取消，则自动退出
    select {
    case <-ctx.Done():
        http.Error(w, "Request timeout", http.StatusGatewayTimeout)
        return
    case result := <-operation:
        // ...
    }
})

// 主函数中发送取消信号
cancel()

```


通过以上步骤，可以在`context`中设置超时时间并控制请求的生命周期，避免因为某个请求无限阻塞而导致整个服务无法正常工作。同时，也可以在主函数中发送取消信号，强制取消各个goroutine的操作，以保证请求的正常结束。


