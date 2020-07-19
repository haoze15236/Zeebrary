# 数据库

## 建表规范

- 必需字段

| 字段                  | 描述                 | 用途                            |
| --------------------- | -------------------- | ------------------------------- |
| REQUEST_ID            | 请求ID               | hap需要                         |
| PROGRAM_ID            | 程序ID               | hap需要                         |
| OBJECT_VERSION_NUMBER | 行版本号，用来处理锁 | 更新数据要在where中加入此行限制 |
| CREATION_DATE         | 创建时间             | 插入数据时取当前时间            |
| CREATED_BY            | 创建人ID             | 插入数据时取当前用户ID          |
| LAST_UPDATED_BY       | 最后更新人ID         | 更新数据时要更新此字段          |
| LAST_UPDATE_DATE      | 最后更新时间         | 更新数据时要更新此字段          |
| LAST_UPDATE_LOGIN     | 最后更新登录人ID     |                                 |

- 可使用[excel表模板](D:\项目\新潮\技术总结\表设计模板1.0.xls)生成表，便于项目管理二开表

- 表字段名使用大写，下划线格式，字段长度参考如下

  | 字段名         | 字段用途             | 字段长度      |
  | -------------- | -------------------- | ------------- |
  | xxx_CODE       | 代码字段             | varchar(10)   |
  | xxx_NAME       | 描述字段             | varchar(30)   |
  | xxx_DESCIPTION | 备注字段             | varchar(2000) |
  | xxx_AMOUT      | 金额字段             | decimal(10,2) |
  | xxx_ID         | 各种id或者number字段 | bigInt(10)    |

  

# Java开发规范

[阿里巴巴Java开发手册](D:\git\doc\阿里巴巴Java开发手册.pdf)

## 空值判断 

```java
//对象判断是否为null
ObjectUtils.isEmpty()
//集合判断是否为null
CollectionUtils.isEmpty()
```

## 常量定义

- 代码中的固定字符一般使用常量池(constants)，建议一个模块定义一个常量池类,根据用途区分成不同interface

```java
  public class K3CloudConstants {
      public static int BATCH_COUNT = 1000;
      /***
       * @Description 接口操作列表
       **/
      public interface OperatorList {
          //单据查询：用于同步数据
          String QUERY = "Kingdee.BOS.WebApi.ServicesStub.DynamicFormService.ExecuteBillQuery";
      }
      /***
       * @Description 查询接口模块
       **/
      public interface QueryModule{
          //金蝶的核算辅助段-共享的系统级维值
          String FND_DIMENSION_VALUE = "FND_DIMENSION_VALUE";
      }
  }
```

  - 可以根据业务使用系统参数方式定义


```java
@Autowired
private ISysParameterService sysParameterService;

this.url = sysParameterService.queryParamValueByCode("CUX_K3CLOUD_INTERFACE_URL", null, null, null, null, null, null, null);
        
```

- <span Style="color:red">禁止写死ID来获取默认值，一般是通过定义Code去匹配，获取ID。</span>



## 项目配置

- jvm启动参数

```shell
-Dfile.encoding=UTF-8 -Duser.region=CN -Duser.language=zh
```



# 接口开发

## 查询其他系统数据

### 伪代码逻辑

```java
syncData(){
    //统一初始化关联数据
    initRefData();
    //请求接口获取请求数据
    List<接口表DTO> reqList;
    BatchOperateUtils.batch(reqList)
    .action(resposeBatchList -> {
        process(resposeBatchList);
        return true;
    })
    .batchCount(1000)
    .debug(true)
    .begin();
    //查询接口表里失效状态数据,为来源系统真实删除的数据，deletList
    deleteBatch(deletList);
}
//业务处理接口数据
public void process(List reqList){
	//查询业务表对应数据,返回
    Map<key,List<业务表DTO>> selList;
    reqList.forEach(x->{
        try{
            if(selList.contain(x.code)){
                x.setId(x.id);
                updateList.add(x);
            }else{
                insertList.add(x);
            }
        }catch(Exception e){
            itfList.add(x);
            return;
        }
    });
	//需要同步数据:updateList需要插入数据:insertList
    updateBatch(updateList);
    insertBatch(insertList);
    //记录到业务接口表
    insertItf(reqList);
}

public void initRefData() {
    //统一修改接口表数据状态为失效
}
//更新业务数据
public void updateBatch(List){
}
//插入业务数据
public void insertBatch(List){
}
//删除业务数据
public void deleteBatch(List){
}
```

### 规范说明

1. 数据量大于1000以上，采用批量处理。
2. 对于存在业务逻辑校验的接口数据同步，建立接口表时要有sync_msg字段，用于记录同步的单条信息业务逻辑上的错误原因，同时不影响其他数据的传输
3. 数据源头系统若存在真实删除数据，可以每次同步之前失效接口表的数据，同步完之后，对接口表里依然失效的数据做逻辑删除(或真实删除)

2. 初始化接口关联数据

   - 全量查询，避免每条数据都请求数据库查询关联数据

   ```java
    //初始化
   IRequest request = RequestHelper.getCurrentRequest(true);
   //部门
   expOrgUnitList = expOrgUnitMapper.selectAll();
   //公司
   fndCompanyList = fndCompanyMapper.selectAll();
   //部门级别
   expOrgUnitLevelList = expOrgUnitLevelMapper.selectAll();
   //部门类型
   expOrgUnitTypeList = expOrgUnitTypeMapper.selectAll();
   ```
   
3. 接口数据映射业务表

   ```java
   employeeBasicList.forEach(employeeBasic -> {
               //------------------数据映射--------------------
               //员工
               ExpEmployee expEmployee = new ExpEmployee();
               expEmployee.setEmployeeId(employeeBasic.getEmployeeId());
               expEmployee.setEmployeeCode(employeeBasic.getEmployeeNo());
               expEmployee.setName(employeeBasic.getEmployeeName());
               expEmployee.setEmail(employeeBasic.getPersonalEmail());
               expEmployee.setMobil(employeeBasic.getPhoneNumber());
               expEmployee.setPhone(employeeBasic.getPhoneNumer());
               }
   ```

4. 更新/插入/删除业务表

   ```java
   //插入集合
   List<CityBasic> insertList = reqList.stream()
       				.filter(reqCity -> !selList.stream()
                       .map(CityBasic::getId)
                       .collect(Collectors.toList())
                       .contains(reqCity.getId())).collect(Collectors.toList());
   //更新集合
   List<CityBasic> updateList = selList.stream()
                       .map(selCity -> reqList.stream()
                               			   .filter(reqCity -> Objects.equals(selCity.getId(),reqCity.getId()))
                               			   .findFirst()
                               			   .map(reqData -> { reqData.setObjectVersionNumber(selCity.getObjectVersionNumber());
                                   return reqData;
                               				})
                               			   .orElse(null))
                       .filter(Objects::nonNull)
                       .collect(Collectors.toList());
   //删除集合
   List<CityBasic> deleteList = selList.stream()
       				.filter(selCity -> !reqList.stream()
                       .map(CityBasic::getId)
                       .collect(Collectors.toList())
                       .contains(selCity.getId()))
       				.collect(Collectors.toList());
   ```

## 发布接口

### 规范

1. 记录入站请求，使用@HapInbound注解，配合前台接口定义功能，可以在前台调用记录里面看到具体的调用记录。
2. 

## 请求外部系统