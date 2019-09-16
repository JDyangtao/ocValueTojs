# ocValueTojs
WKWebView与oC交互
###1、UIWebViewOC传值给js
- 传值
```
#pragma mark - 图标H5数据交互
-(void)webViewDidStartLoad:(UIWebView *)webView
{
    self.jsContext = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    // 把数据传递给 JS。本质上就是让 JS 调用一个返回值的 OC 方法。
    JDWeakSelf
    self.jsContext[@"backDataToJS"] = ^{
        return weakSelf.dict;
    };
    self.jsContext.exceptionHandler = ^(JSContext *context, JSValue *exceptionValue) {
        context.exception = exceptionValue;
    };
}
```
- 接收
```
  //这个是OC传过来的值
   var data = backDataToJS();
```
###2、WKWebViewOC传值给js
- 传值
```
-(void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
    //sendKey是和js约定的方法，我是OC是要传的值
    //NSString * jsStr = [NSString stringWithFormat:@"sendKey('%@')", @"我是OC"];
    NSString * jsStr = [NSString stringWithFormat:@"sendKey('%@','%@','%@')", @"kobe",@"123456",@"iphone 6"];
    [self.webView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        
    }];
}
```
- 接收
```
//js接收OC的传值,实现约定sendKey方法，name,token,device为参数
 function sendKey(name,token,device){
        document.getElementById("demo").innerHTML = token;
}
```
###3、OC获取js点击事件
- 遵守协议
```
@interface JDControlOvervView()<WKScriptMessageHandler,WKUIDelegate,WKNavigationDelegate>
@end
```
- 设置代理
```
    self.webView.UIDelegate = self;
    self.webView.navigationDelegate = self;
    [[self.webView configuration].userContentController addScriptMessageHandler:self name:@"displayDate"];
```
- 实现代理
```
#pragma mark - 代理方法
-(void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    NSLog(@"==message==%@",message.body[@"name"]);
}
```
- js代码
```
function displayDate() {
            document.getElementById("demo").innerHTML = Date();
            let dict = {"name":"123456"};
            window.webkit.messageHandlers.displayDate.postMessage(dict);
        }
```
###4、WKWebView出现循环引用问题
-  [[self.webView configuration].userContentController addScriptMessageHandler:self name:@"displayDate"];方法出现循环用问题
```
- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];
    // 因此这里要记得移除handlers
    [self.controlOverView.webView.configuration.userContentController removeScriptMessageHandlerForName:@"displayDate"];
}
```
