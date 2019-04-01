# CURD

## C

| 英文   | 中文 | SQL    | HTTP     |
| ------ | ---- | ------ | -------- |
| Create | 建立 | INSERT | POST |


### Controller 

```java
/**
 * 新增请求使用POST方法
 * 参数放入request body
 * 重要参数进行valid校验
 * Content-Type = application/json
 */
@PostMapping("/user")
public Wrapper<Boolean> insertUser(@RequestBody @Valid UserDTO userDTO) {
    return new Wrapper<>(userService.insertUser(userDTO));
}
```

URL：**Restful 格式**

注解：使用 **Post 方法**，`@PostMapping()`(相当于`@RequestMapping(method = RequestMethod.POST)`)。

参数：要求使用以 **DTO 结尾**的对象来接收前端参数，重要的参数需要使用`@Valid`进行校验。

注释：必须包含**方法的说明，参数说明，返回值说明**。

命名：**`insert/add`开头, 方法的命名与功能必须一致**。


### Service 

```java
/**
 * 新增用户
 *
 * @param userDTO 用户信息
 * @return true 新增成功
 */
@Override
public Boolean insertUser(UserDTO userDTO) {
    UserPO userPO = UserPO.of()
            .setName(userDTO.getUserName())
            .setPortrait(userDTO.getPortrait())
            .setCreateTime(new Date())
            .setUpdateTime(new Date());

    int result = userDao.insertUser(userPO);
    if (result < 1) {
        log.error("失败的新增数据 => {}", JSONObject.toJSONString(userPO));
        throw ExceptionConstructor.runtimeBusExpConstructor(ExpCodeEnum.INSERT_USER_FAIL);
    }
    return true;
}
```

数据转换：视图与 Service 层交互使用 DTO，数据与持久层交互时，使用 PO，所以在数据保存到持久层前需要做** DTO -> PO 的转换**。

异常：根据具体业务引入对应的异常。

注释：**方法说明，参数说明，返回值说明**。

日志：**异常分支必须打印异常**，适当增加能帮助定位问题的异常。

命名：**`insert/add`开头, 方法的命名与功能必须一致**。


### Dao

```java
/**
 * 新增用户信息
 *
 * @param userPO 用户信息
 * @return 插入数目
 */
int insertUser(@Param("userPO") UserPO userPO);
```

注释：**方法说明，参数说明，返回值说明**。

命名：**`insert/add`开头, 方法的命名与功能必须一致**。

数据：**参数必须接收以 PO 结尾的对象，返回值为插入数目**。


## R

| 英文 | 中文 | SQL    | HTTP |
| ---- | ---- | ------ | ---- |
| Read | 读取 | SELECT | GET  |

### Controller 

```java
/**
 * 获取请求使用GET方法
 * 非必选查询使用对象接收
 * Content-Type = application/x-www-form-urlencoded
 */
@GetMapping("/users")
public Wrapper<UserPaginationVO> queryUserList(UserQO userQO) {
    return new Wrapper<>(userService.queryUserList(userQO));
}

/**
 * 获取请求使用GET方法
 * 确定具体资源的条件放入URL
 * Content-Type = application/x-www-form-urlencoded
 */
@GetMapping("/user/{userName}")
public Wrapper<UserVO> queryUserByName(@PathVariable("userName") String userName) {
    return new Wrapper<>(userService.queryUserByName(userName));
}
```

URL：**Restful 格式**，列表使用复数，单个资源使用单数。

注解：使用 **Get 方法**，`@GetMapping()`(相当于`@RequestMapping(method = RequestMethod.GET)`)

参数：**可以直接从 URL 中获取，也可以用对象接受，如果使用对象，必须以 QO 结尾，`Content-Type=application/x-www-form-urlencoded`**。

注释：必须包含**方法的说明，参数说明，返回值说明**。

命名：**`query`开头, 方法的命名与功能必须一致**。


### Service

```java
public UserVO queryUserByName(String userName) {
    UserPO userPO = userDao.queryUserByName(userName);

    if (ObjectUtils.isEmpty(userPO)) {
        throw ExceptionConstructor.runtimeBusExpConstructor(ExpCodeEnum.USER_OBJ_EMPTY, userName);
    }
    return UserVO.of()
            .setUserName(userPO.getName())
            .setPortrait(CommonUtils.generateFileUrl(userPO.getPortrait(), fileDomain))
            .setCreateTime(userPO.getCreateTime());
}
```

数据转换：**如果参数是对象数据与持久层交互时需要做 QO -> PO 的转换**，返回结果需要做 PO -> VO 的转换。

异常：根据具体业务引入对应的异常。

注释：**方法说明，参数说明，返回值说明**。

日志：**异常分支必须打印异常**，适当增加能帮助定位问题的异常。

命名：**`query`开头（如果是计算查询以`count`开头）, 方法的命名与功能必须一致**。


### Dao

```java
/**
 * 根据用户名精确查询用户
 *
 * @param name 用户名
 * @return UserPO
 */
UserPO queryUserByName(@Param("name") String name);
```

注释：**方法说明，参数说明，返回值说明**。

命名：**`query`开头（如果是计算查询以`count`开头), 方法的命名与功能必须一致**。

数据：**返回值如果是表对象，必须用 PO 对象接收，如果是多表联合结果集，使用 DTO 对象接收**。



## U

| 英文   | 中文 | SQL    | HTTP     |
| ------ | ---- | ------ | -------- |
| Update | 更新 | UPDATE | PUT |

### Controller 

```java
/**
 * 修改请求使用PUT方法
 * 定位资源的参数放入URL
 * 参数放入request body
 * 重要参数进行valid校验
 * Content-Type = application/json
 */
@PutMapping("/user/{userName}")
public Wrapper<Boolean> updateUserByName(@PathVariable("userName") String userName, @RequestBody @Valid UserDTO userDTO) {
    return new Wrapper<>(userService.updateUserByName(userName, userDTO));
}
```

URL：**Restful 格式**

注解：使用 **PUT 方法**，`@PutMapping()`(相当于`@RequestMapping(method = RequestMethod.PUT)`)。

参数：**可以直接从 URL 中获取，也可以用对象接受，如果使用对象，必须以DTO结尾**。

注释：必须包含**方法的说明，参数说明，返回值说明**。

命名：**`update`开头, 方法的命名与功能必须一致**。


### Service

```java
/**
 * 根据用户名修改用户信息
 *
 * @param userName 用户名
 * @param userDTO  新的用户信息
 * @return true 修改成功
 */
@Override
public Boolean updateUserByName(String userName, UserDTO userDTO) {
    UserPO userPO = UserPO.of()
            .setName(userDTO.getUserName())
            .setPortrait(userDTO.getPortrait())
            .setCreateTime(new Date())
            .setUpdateTime(new Date());

    int result = userDao.updateUserByName(userName, userPO);
    if (result < 1) {
        log.error("失败的更新数据 => {}", userName);
        log.error("失败的更新数据 => {}", JSONObject.toJSONString(userPO));
        throw ExceptionConstructor.runtimeBusExpConstructor(ExpCodeEnum.UPDATE_USER_FAIL);
    }
    return true;
}
```

数据转换：视图与 Service 层交互使用 DTO，数据与持久层交互时，使用 PO，所以在数据保存到持久层前需要做** DTO -> PO 的转换**。

异常：根据具体业务引入对应的异常。

注释：**方法说明，参数说明，返回值说明**。

日志：**异常分支必须打印异常**，适当增加能帮助定位问题的异常。

命名：**`update`开头, 方法的命名与功能必须一致**。



### Dao

```java
/**
 * 根据用户名更新用户
 *
 * @param name   用户名
 * @param userPO 更新用户信息
 * @return 更新数目
 */
int updateUserByName(@Param("name") String name, @Param("userPO") UserPO userPO);
```

注释：**方法说明，参数说明，返回值说明**。

命名：**`update`开头, 方法的命名与功能必须一致**。

数据：**参数必须接收以 PO 结尾的对象，返回值为更新数目**。



## D

| 英文   | 中文 | SQL    | HTTP   |
| ------ | ---- | ------ | ------ |
| Delete | 删除 | DELETE | DELETE |

### Controller 

```java
/**
 * 删除请求使用DELETE方法
 * 定位资源的参数放入URL
 * Content-Type = application/x-www-form-urlencoded
 */
@DeleteMapping("/user/{userId}")
public Wrapper<Boolean> deleteUserById(@PathVariable("userId") Integer userId) {
    return new Wrapper<>(userService.deleteUserById(userId));
}
```

URL：**Restful 格式**

注解：使用 **Delete 方法**，`@GetMapping()`(相当于`@RequestMapping(method = RequestMethod.DELETE)`)。

参数：直接从 URL 中获取。

注释：必须包含**方法的说明，参数说明，返回值说明**。

命名：**`delete`开头, 方法的命名与功能必须一致**。



### Service

```java
/**
 * 根据用户Id删除用户
 *
 * @param userId 用户Id
 * @return true 删除成功
 */
@Override
public Boolean deleteUserById(Integer userId) {
    int result = userDao.deleteUserById(userId);
    if (result < 1) {
        log.error("失败的删除数据 => {}", userId);
        throw ExceptionConstructor.runtimeBusExpConstructor(ExpCodeEnum.DELETE_USER_FAIL);
    }
    return true;
}
```

异常：根据具体业务引入对应的异常。

注释：**方法说明，参数说明，返回值说明**。

日志：**异常分支必须打印异常**，适当增加能帮助定位问题的异常。

命名：**`delete`开头, 方法的命名与功能必须一致**。



### Dao

```java
/**
 * 根据用户Id删除用户
 *
 * @param id 用户Id
 * @return 删除数目
 */
int deleteUserById(@Param("id") Integer id);
```

注释：**方法说明，参数说明，返回值说明**。

命名：**`delete`开头, 方法的命名与功能必须一致**。

数据：**返回值为删除数目**。



## 存放路径
```
--dao

    --XXXDao接口

    --mapper

        ​--xxxMapper.xml
```

注意：

1. 配置`MsDemoApplication.java`: `@MapperScan({"com.tv.**.dao"})`。
2. 配置`application.yml`: `mybatis.mapper-locations: classpath*:com/tv/ms/**/mapper/*.xml`。



## POJO特殊说明

**PO** :  **Persistant Object**, 持久对象，与数据库中的表相映射的`java`对象，**字段必须与数据库表一一对应**。

**如果查询返回值字段是通过多个表连接得到的，需要重新定义一个多字段的 DTO**。

通过注解`@Data(staticConstructor = "of")`和`@Accessors(chain = true)`可极大简化代码量。

