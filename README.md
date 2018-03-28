# BurpSuite插件笔记

此文档用来记录学习开发burp插件中的一些笔记，官方在线API文档：**https://portswigger.net/burp/extender/api/**

## API分类


1. #### 插件入口类：

    | 接口 | 描述 |
    | - | -: |
    | IBurpExtender | 扩展类 |
    | IBurpExtenderCallbacks | 扩展回调对象 |

2. #### UI相关类：

    | 控件 | 接口 | 描述 |
    | - | :-: | -: |
    | Burp标签页 | Itab | Burp标签 |
    | 菜单 | IContextMenuFactory | 上下文菜单工厂|
    |      | IContextMenuInvocation | 上下文菜单调用 |
    | 消息编辑器 | IMessageEditorTabFactory | 消息编辑器标签工厂 |
    |           | IMessageEditorTab | 消息编辑器标签 |
    |           | IMessageEditor | 信息编辑器 |
    |           | IMessageEditorController | 信息编辑器的控制器 |
    | 文本编辑器 | ITextEditor| 文本编辑器 |

3. #### 工具套件类

    | 套件 | 接口 | 描述 |
    | - | :-: | -: |
    | 范围 (Scope) | IScopeChangeListener | 范围改变监听器 |
    | 代理 (Proxy) | IProxyListener | 代理监听器 |
    |      | IInterceptedProxyMessage | 代理拦截信息 |
    | 入侵者 (Intruder) | IIntruderAttack | 入侵者攻击信息 |
    |           | IIntruderPayloadGenerator | Payload成生器 |
    |           | IIntruderPayloadGeneratorFactory | payload工厂 |
    |           | IIntruderPayloadProcessor | Paylaod处理器 |
    | 扫描 (Scanner) | IScannerListener| 扫描监听器 |
    |                | IScanIssue | 扫描报告 |
    |                | IScannerCheck | 扫描检查 |
    |                | IScannerInsertionPoint | 扫描插入点 |
    |               | IScannerInsertionPointProvider | 扫描插入点提供者 |
    |               | IScanQueueItem | 扫描队列条目 |
    | 会话 (session) | ISessionHandlingAction | 会话处理动作 |


4. #### HTTP消息处理接口类

    | 接口 | 描述 |
    | - | -: |
    | IHttpListener | HTTP消息监听器 |
    | IHttpRequestResponse | 请求响应消息 |
    | IHttpRequestResponsePersisted | 请求响应信息维持 |
    | IHttpRequestResponseWithMarkers | 请求响应信息标记? |
    | - | - |
    | IRequestInfo | HTTP服务消息 |
    | IResponseInfo | 请求信息 |
    | IHttpService | 响应消息 |
    | Icookie | cookie |
    | Iparameter | 参数 |

5. #### 辅助类

    | 接口 | 描述 |
    | - | -: |
    | IExtensionHelpers | 辅助类 |

6. #### 其他

    | 接口 | 描述 |
    | - | -: |
    | ITempFile | 缓存文件 |
    | IExtensionStateListener | 扩展状态监听 |



#### 并从中分出2个小类：四大监听器和四大工厂:

1. #### 四大监听器

    | 监听器 | 描述 |
    | - | -: |
    | IHttpListener | Http消息监听器 |
    | IProxyListener | 代理监听器 |
    | IScannerListener | 扫描监听器 |
    | IExtensionStateListener | 扩展状态监听器 |
    | IScopeChangeListener | 范围改变监听器 |

2. #### 四大工厂

    | 工厂 | 生成器 | 处理器 |
    | - | :-: | -: |
    | IContextMenuFactory | IContextMenuInvocation |  |
    | IMessageEditorTabFactory | IMessageEditorTab |  |
    | IIntruderPayloadGeneratorFactory | IIntruderPayloadGenerator | IIntruderPayloadProcessor |
    | IScannerInsertionPointProvider | IScannerInsertionPoint |  |


## API

* ### IBurpExtender  

    所有拓展都必须实现这个接口，并且实现的类名必须叫“BurpExtender”，存在于burp包中，
    IBurpExtender中只有一个方法需要实现:
    * void registerExtenderCallbacks(IBurpExtenderCallbacks callbacks)
    
    当拓展被加载时自动调用此方法,也就是说这就是我们的入口

    参数为IBurpExtenderCallbacks接口的实例callbacks


* ### IBurpExtenderCallbacks

    Burp Suite使用这个接口传递给扩展一组回调方法，扩展可以使用这些方法在Burp内执行各种操作.

    一些可能用到的方法.

    * java.io.OutputStream getStdout()

    此方法用于获取当前拓展的标准输出流。扩展应该将所有输出写入此流，允许Burp用户配置如何从UI内处理输出.

    * java.io.OutputStream getStderr()

    此方法用于获取当前拓展的标准错误流。扩展应该将所有错误消息写入此流，允许Burp用户配置如何从UI中处理输出.

   * void setExtensionName(java.lang.String name)

    此方法用于设置当前拓展的显示名称，该名称将显示在扩展器工具的用户界面中.

    * IExtensionHelpers getHelpers()

    此方法用于获取IExtensionHelpers对象，该扩展名可用于执行许多有用的任务.

    * void registerHttpListener(IHttpListener listener)

    这个方法用来注册一个监听器，这个监听器将被通知任何Burp工具所做的请求和响应。扩展可以通过注册HTTP侦听器来执行自定义分析或修改这些消息

* ### IExtensionHelpers

    这个接口包含了一些辅助方法，这些方法可以用来协助Burp扩展中出现的各种常见任务。扩展可以调用IBurpExtenderCallbacks.getHelpers来获得这个接口的一个实例.
    * IRequestInfo analyzeRequest(byte[] request)

    用于解析HTTP请求,并获得关于该请求的详细信息,返回IRequestInfo对象

    * IResponseInfo analyzeResponse(byte[] response)

    用于解析HTTP响应,并获得关于该响应的详细信息,返回IResponseInf对象

    * byte[] buildHttpMessage(java.util.List\ headers, byte[] body)

    这个方法构建一个包含指定头和消息主体的HTTP消息。如果适用，Content-Length头将根据正文的长度添加或更新






























* ### 参考连接 ：
    http://gv7.me/articles/2017/classification-of-burp-apis/

    http://03i0.com/2017/11/17/%E7%BC%96%E5%86%99%E7%AC%AC%E4%B8%80%E4%B8%AABurpsuite%E6%8F%92%E4%BB%B6/
