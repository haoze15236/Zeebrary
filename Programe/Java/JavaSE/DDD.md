

# api

>  用户接口层，向外提供服务

- controller：提供资源服务，XxxController.java
- dto：数据传输对象，XxxDTO.java，对于一些复杂页面需要多个实体组合时，可使用DTO对象来传输数据。

# app

> 应用层，包含应用服务，负责用例流程调度，事务控制

- service：应用服务接口，XxxService.java，应用服务里进行事务控制，流程调度。
- service.impl：应用服务实现，XxxServiceImpl.java 。在处理业务流程，如果使用设计模式，则接口和抽象类放在此
- assembler：DTO组装器，XxxAssembler.java，复杂DTO的组装，简单的直接使用Entity即可

# domain

> domain：领域层，包含领域对象和领域服务，专注核心业务

- entity：实体对象，与表做映射，具备一些简单的自治的业务方法
- service：领域服务，命名一般按提供的业务功能命名，通常用于封装一个领域内的复杂业务逻辑，简单的业务逻辑在 app 层完成即可，不需要领域层。
- repository：资源库接口，XxxRepository.java，提供数据资源的操作方法，如数据库增删改查、Redis增删改查等，查询操作建议写到 repository 内。
- vo：值对象，XxxVO.java，领域内用到的数据封装，对于一些没有实体对象的数据对象但又在领域中用到，使用值对象封装

# infra

> 基础设施层，提供数据持久化、防腐层实现、第三方库、消息等

- mapper：Mapper接口，XxxMapper.java
- repository.impl：资源库实现，XxxRepositoryImpl.java，业务一定不要侵入到这里
- constant：常量
- util：工具

