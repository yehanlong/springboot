(1)从spring.factories配置文件中加载自动配置类；

(2)加载的自动配置类中排除掉@EnableAutoConfiguration注解的exclude属性指定的自动配置类；

(3)然后再用AutoConfigurationImportFilter接口去过滤自动配置类是否符合其标注注解（若有标注的话）@ConditionalOnClass,@ConditionalOnBean和@ConditionalOnWebApplication的条件，若都符合的话则返回匹配结果；

(4)然后触发AutoConfigurationImportEvent事件，告诉ConditionEvaluationReport条件评估报告器对象来分别记录符合条件和exclude的自动配置类。

(5)最后spring再将最后筛选后的自动配置类导入IOC容器中