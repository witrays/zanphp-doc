ExceptionHandler
================

文件位置
~~~~~~~~

::

    resource/middleware/exceptionHandler.php

功能
----

自定义请求异常处理函数，当请求处理过程中抛出异常时，异常可以经过中间件处理后透传给调用方。

配置示例
--------

tcp
~~~

.. code:: php

    <?php
    use Zan\Framework\Network\Tcp\Exception\Handler\GenericExceptionHandler;

    return [
        'match' => [
            [
                "/com/youzan/nova/framework/generic/service/GenericService/invoke", "genericExceptionHandlerGroup",
            ],
            [
                "/Com/Youzan/Nova/Framework/Generic/Php/Service/GenericTestService/ThrowException", "genericExceptionHandlerGroup",
            ],
            [
                ".*", "all"
            ]
        ],
        'group' => [
            "genericExceptionHandlerGroup" => [
                GenericExceptionHandler::class
            ],
            "all" => [
                GenericExceptionHandler::class
            ],
        ],
    ];

-  match用于服务分组，服务名称支持正则表达式匹配。
-  group设置分组名和对应的异常处理器，异常处理器需要实现接口Zan\Framework\Contract\Network\RequestFilter，一个分组名可以设置多个处理器。

异常处理器的实现示例为：

.. code:: php

    <?php
    use Zan\Framework\Contract\Foundation\ExceptionHandler;

    class GenericExceptionHandler implements ExceptionHandler
    {
        public function handle(\Exception $e)
        {
            sys_error("GenericExceptionHandler handle: ".$e->getMessage());
            throw new \Exception("网络错误", 0);
        }
    }

handle对异常进行处理，加工处理后需要透传给调用方的异常再次抛出或返回即可，返回异常采用函数或生成器均可。

http
~~~~

http配置内容与tcp类似，不同点在于tcp以服务名和方法名为key，http以url的path为key（path未指明默认为/index/index/index）来匹配分组，http示例为：

.. code:: php

    <?php
    return [
         'match' => [
             [
                "index/index/index",  "all",
             ],
             [
                ".*", "all"
             ]
         ],
         'group' => [
             "all" => [
                 TestExceptionHandler::class,
             ],
         ],
    ];

 异常处理器的实现示例为：

.. code:: php

    <?php
    use Zan\Framework\Contract\Foundation\ExceptionHandler;

    class TestExceptionHandler implements ExceptionHandler
    {
        public function handle(\Exception $e)
        {
            // 针对异常自行判断决定是否
            return new Response("网络错误");
            // or 
            return null; // 不做处理
        }
    }

handle对异常进行处理，加工处理后需要透传给调用方的异常需要返回，返回异常采用函数或生成器均可。

针对常用的http
exception，zan框架内置了通用的异常处理逻辑，包括Redirect、PageNotFound等，业务只需要在逻辑代码中抛出对应类型的异常即可触发对应的处理逻辑。

-  url重定向：

抛出RedirectException异常，异常实例中redirectUrl字段指定重定向跳转url

-  Page Not Found

抛出PageNotFoundException异常，页面重定向到resource/config/$ENV/error.php中配置404的url地址，error.php示例为：

.. code:: php

    <?php
    return [
        '404' => url
    ]

-  Forbidden

抛出TokenException异常，当http请求头部Accept字段指定application/json时，返回异常的json串，便于调试，否则返回zan/src/Foundation/View/Pages/Error.php页面

-  InvalidRoute

路由不合法，抛出InvalidRouteException异常，页面重定向到resource/config/$ENV/error.php中配置404的url地址

-  业务异常

抛出:raw-latex:`\Exception基类异常`，异常码范围是10000-60000，根据http请求头部Accept字段返回响应，与Forbidden的处理方式相同

-  Server Unavailable

服务不可用，抛出:raw-latex:`\Exception基类异常`，异常码为503，根据http请求头部Accept字段返回响应，与Forbidden的处理方式相同




