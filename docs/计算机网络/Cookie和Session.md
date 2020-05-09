- ## Cookie和Session：

  **cookie简介：**

  - 是由服务器发给客户端的特殊信息，以文本的形式存放在客户端
  - 客户端再次请求的时候，会把cookie回发
  - 服务器接收到后，会解析cookie生成与客户端相对应的内容

  **cookie的设置以及发送过程：**

  ![image-20191108114910769](images/image-20191108114910769.png)

  **session简介：**

  - 服务器端的机制，在服务器上保存的信息
  - 解析客户端请求并操作session id，按需保存状态信息

  **两者的区别：**

  - cookie数据存放在客户的浏览器上，session数据放在服务器上
  - session相对于cookie更安全
  - 若考虑减轻服务器负担，应当使用cookie