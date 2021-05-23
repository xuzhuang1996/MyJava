1. OperationLog接口，定位：日志操作的字段值，包括：用户Id、来源、目标、IP等，可以包括prepare操作。
2. AbstractOperationLog，实现OperationLog接口。定位：实现日志的统一处理，包括协调正确日志、错误日志等信息输出。包含2个属性：Object[] reqArgs参数列表、返回对象Object response。
3. Aspect。切点。当该注解放置在控制器的方法上时生效。
   1. 初始化一个操作日志对象。根据方法的入参进行构造操作日志子类实例。

            private AbstractOperationLog createOpLogBean(Class<? extends AbstractOperationLog> opLogClazz, Object[] reqArgs, Object response) {
                // 获取操作日志子类的构造函数
                Constructor<? extends AbstractOperationLog> constructor = ClassUtils.getConstructorIfAvailable(opLogClazz);
                Assert.notNull(constructor, "default Constructor not found!");
                // 构造操作日志子类实例
                AbstractOperationLog opLogInstance = BeanUtils.instantiateClass(constructor);
                Assert.notNull(opLogInstance, "OperationLog instantiate failed!");
                opLog.setReqArgs(reqArgs);
                // 将子类注入到spring。当子类spring容器的bean时，可以直接使用
                applicationContext.getAutowireCapableBeanFactory().autowireBean(opLogInstance);

                return opLogInstance;
            }
4. 执行控制器方法

            Object response = joinPoint.proceed();
            opLog.setResponse(response);
5. 将操作日志实例的方法返回值，转换为平台接口Body体，发送http请求

            OperLogInfo operLogInfo = convert(opLog);
            OperationLogUtils.writeLog(operLogInfo, header)