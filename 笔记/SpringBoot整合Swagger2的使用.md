# 1. 依赖

````.java

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

````



# 2.配置类

````.java
@RestController
@Api(tags = "用户管理相关接口")
@RequestMapping("/user")
public class UserController {

    @PostMapping("/")
    @ApiOperation("添加用户的接口")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "username", value = "用户名", defaultValue = "李四"),
            @ApiImplicitParam(name = "address", value = "用户地址", defaultValue = "深圳", required = true)
    }
    )
    public RespBean addUser(String username, @RequestParam(required = true) String address) {
        return new RespBean();
    }

    @GetMapping("/")
    @ApiOperation("根据id查询用户的接口")
    @ApiImplicitParam(name = "id", value = "用户id", defaultValue = "99", required = true)
    public User getUserById(@PathVariable Integer id) {
        User user = new User();
        user.setId(id);
        return user;
    }
    @PutMapping("/{id}")
    @ApiOperation("根据id更新用户的接口")
    public User updateUserById(@RequestBody User user) {
        return user;
    }
}
————————————————
版权声明：本文为CSDN博主「_江南一点雨」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u012702547/article/details/88775298


````

# 3 swagger详细内容

常用注解：
- @Api()用于类；
  表示标识这个类是swagger的资源

- @ApiOperation()用于方法；
  表示一个http请求的操作,可以写该请求所实现的功能

  ```java
  name/value: Path parameters must always be named as the path section they represent
  ```

- @ApiParam()用于方法，参数，字段说明；
  表示对参数的添加元数据（说明或是否必填等）

- @ApiModel()用于类
  表示对类进行说明，用于参数用实体类接收

- @ApiModelProperty()用于方法，字段
  表示对model属性的说明或者数据操作更改

- @ApiIgnore()用于类，方法，方法参数
  表示这个方法或者类被忽略

- @ApiImplicitParam() 用于方法
  表示单独的请求参数

- @ApiImplicitParams() 用于方法，包含多个 @ApiImplicitParam

具体使用举例说明：
@Api()
用于类；表示标识这个类是swagger的资源
tags–表示说明
value–也是说明，可以使用tags替代
但是tags如果有多个值，会生成多个list

----

## Api

  用在Controller中，标记一个Controller作为swagger的文档资源

- value	Controller的注解
- description	对api资源的描述
- hidden	配置为true 将在文档中隐藏

## ApiOperation

  该注解用在Controller的方法中，用于注解接口

- value	接口的名称
- notes	接口的注释
- response	接口的返回类型，比如说：response = String.class
- hidden	配置为true 将在文档中隐藏

## ApiParam

  该注解用在方法的参数中。

- name	参数名称
- value	参数值
- required	是否必须，默认false
- defaultValue	参数默认值
- type	参数类型
- hidden	隐藏该参数
    使用方法：

    @ApiOperation(value = "添加权限",notes = "插入权限",response = JsonData.class)
    @PostMapping("/insertAcl.json")
    public JsonData insertAcl(@ApiParam(name = "param",value = "实体类AclParam",required = true) AclParam param){
    
      该注解用在Controller的方法中，用于注解方法的返回状态。
## ApiResponses/ApiResponse

属性名称	说明
code	http的状态码
message	状态的描述信息
response	状态相应，默认响应类 Void
  使用方法：

    @ApiOperation(value = "菜单",notes = "进入菜单界面",nickname = "菜单界面")
    @ApiResponses({
            @ApiResponse(code = 200,message = "成功！"),
            @ApiResponse(code = 401,message = "未授权！"),
            @ApiResponse(code = 404,message = "页面未找到！"),
            @ApiResponse(code = 403,message = "出错了！")
    })
    @GetMapping("/aclModule.page")
    public ModelAndView aclModule(Model model){
    }
## ApiModel

  该注解用在实体类中。

属性名称	说明
value	实体类名称
description	实体类描述
parent	集成的父类，默认为Void.class
subTypes	子类，默认为{}
reference	依赖，默认为“”
  使用方法：

````
@ApiModel(value = "JsonData",description = "返回的数据类型")
public class JsonData {
}
````

## ApiModelProperty

  该注解用在实体类的字段中。

属性名称	说明
name	属性名称
value	属性值
notes	属性注释
dataType	数据类型，默认为“”
required	是否必须，默认为false
hidden	是否隐藏该字段，默认为false
readOnly	是否只读，默认false
reference	依赖，，默认“”
allowEmptyValue	是否允许空值，默认为false
allowableValues	允许值，默认为“”
  使用方法：

    //返回状态信息
    @ApiModelProperty(name = "code",value = "状态code",notes = "返回信息的状态")
    private int code;
    //返回携带的信息内容
    @ApiModelProperty(name = "msg",value = "状态信息",notes = "返回信息的内容")
    private String msg = "";
    //返回信息的总条数
    @ApiModelProperty(name = "count",value = "查询数量",notes = "返回信息的条数")
    private int count;
    //返回对象
    @ApiModelProperty(name = "data",value = "查询数据",notes = "返回数据的内容")
    private Object data;


## ApiImplicitParams/ApiImplicitParam

  该注解用在Controller的方法中，同ApiParam的作用相同，但是比较建议使用ApiParam。

属性名称	说明
name	参数名称
value	参数值
defaultValue	参数默认值
required	是否必须
allowMultiple	是否允许重复
dataType	数据类型
paramType	参数类型
  使用方法：

    @ApiOperation(value = "创建用户",notes = "根据User对象创建用户")
    @ApiImplicitParam(name = "user",value = "用户详细实体user")
    @RequestMapping(value="/", method=RequestMethod.POST)
    public String postUser(@ModelAttribute User user){
    }
