#一、用户认证相关

###1.用户注册：

1. **URL:**

http://localhost:8080/oauth/register



2. **发送值：**

BODY下发送JSON对象---对应服务器的LoginPara实例类。

+ *userName：*	必选项  用客户端判断是否合规

+ *passWord:*     必选项  用客户端判断是否合规

+ *clientId：*        必选项

  

3. **返回值：**

      返回一个resultMsg的JSON字串。下面就直接说resultMsg中3个属性值的意思了

+ *int errcod*
  + *ResultStatusCode.INVALID_CLIENTID.getErrcode()* 	 说明不是自家客户端
  +  *ResultStatusCode.USERALREADY_REGISTERED.getErrcode()*     帐号已经被注册过了
  + *ResultStatusCode.SYSTEM_ERR.getErrcode()*           服务端异常
  + *ResultStatusCode.OK.getErrcode()*           说明注册成功
+ *String errmsg*
+ *Object p2pdata;*

**客户端注册事项**
  1.客户端是首先要有 audienceEntity.getClientId() 这个客户端标识的。否则服务器是不会给你token的
  2.客户端的密码是要事先进行MyUtils.getMD5(明文+"LaoSiJi")加密的。
    数据库中的密码==  MD5（明文+"LaoSiJi"MD5加密 + salt）。注册的时候要这么做啊。

---

###2.用户登录：

1. **URL:** 

		 http://localhost:8080/oauth/token



2. **发送值：**

		  BODY下发送JSON对象---对应服务器的LoginPara实例类。

+ *clientId*：<u>必选项  为空或者不正解 返回 INVALID_CLIENTID</u>
+ *userName：*<u>必选项  为空直接返回 NVALID_PASSWORD</u>
+ passWord;       <u>必选项  对原始密码进行加盐MD5加密MyUtils.getMD5(loginPara.getPassWord()+user.</u> <u>getSalt())再比对，错误的话返回INVALID_PASSWORD   这里的user对象是通过数据库里查询的</u>

  

3. **返回值：**

    获取ResultMsg对象。同注册一样有3个属性值，这里就讲其中最后一个属性为AccessToken对像，包含：

    `String access_token`			//jwt值
    `private String token_type`		//token的类型

    `private long expires_in`		//过期时长

     `private String refresh_token`;		//刷新token时使用的jwt

**int errcod**
   	ResultStatusCode.INVALID_CLIENTID.getErrcode() 说明不是自家客户端
	ResultStatusCode.INVALID_PASSWORD.getErrmsg() 用户不存在或者密码错误
	ResultStatusCode.SYSTEM_ERR.getErrcode() 服务端异常
	ResultStatusCode.OK.getErrcode()  登录成功并且会返回token值

---

###3.API jwt验证

1. **URL:** 

这里不要求，就是API本身的URL。

2. **请求值：**

​     http请求统一要在客户端要在HTTP访问头中加入Authorization="bearer 本地保存的访问jwt"，客户端采用
     okHttp的拦截器进行统一封装，更改请求头部。使用ApiHttpClient.getInstance()获得http客户端。

3.  **返回值：**

​	1.jwt正常一切OK，认为完全忽略整个过程。该返回什么就返回什么。
	2.jwt过期或出错返回ResultMsg消息
int errcod
   ResultStatusCode.INVALID_TOKEN.getErrcode() 验证不通过

   ResultStatusCode.EXPIRES_TOKEN.getErrcode() jwt过期，并且会在头部Authorization写入"expires"



###4.jwt过期换取新token

1. **URL:** 

 http://localhost:8080/oauth/refreshToken

2. **请求值：**

​	请求头Authorization值为 "bearer " + refreshJwt
	表单值：clientId:客户端识别号

3. **返回值：**

​	`1.body值 `               error clientId  客户端不识别
	`2.ResultMsg对象`
		ResultStatusCode.OK.getErrcode() 获取成功，并会携带包含双jwt的AccessToken对象
		ResultStatusCode.EXPIRES_TOKEN.getErrcode() 说明refresh-jwt也过期了，并且会在头部Authorization写入"expires"
		ResultStatusCode.SYSTEM_ERR.getErrcode() 验证出错有异常











经用户名密码登录后就会从服务器获得，Token保存到本地，之后访问API服务器可以对指定的访问过虑检查，所以
  客户端要在HTTP访问头中加入Authorization="bearer jwt"，如果能够正常解析，就放行访问。如果有问题不正解的
  就会返回ResultMsg带INVALID_TOKEN标识。如果是Token超时的 就会返回Authorization="expires"头部标记的信息。





//使用FastJson去解析Json数据，将json字符串转换成一个ResultBeanData对象
         ResultBeanData resultBeanData = JSON.parseObject(json,ResultBeanData.class);
         resultBean = resultBeanData.getResult();
         Log.e(TAG,"解析成功=="+resultBean.getHot_info().get(0).getName());

/申明给服务端传递一个json串
 //创建一个OkHttpClient对象
 OkHttpClient okHttpClient = new OkHttpClient();
 //创建一个RequestBody(参数1：数据类型 参数2传递的json串)
 RequestBody requestBody = RequestBody.create(JSON, json);
 //创建一个请求对象
 Request request = new Request.Builder()
   .url("http://192.168.0.102:8080/TestProject/JsonServlet")
   .post(requestBody)
   .build();
 //发送请求获取响应
 try {
 Response response=okHttpClient.newCall(request).execute();
  //判断请求是否成功
  if(response.isSuccessful()){\
   //打印服务端返回结果
   Log.i(TAG,response.body().string());

  }