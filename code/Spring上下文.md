### 获取Bean
```java
@Configuration
public class ApplicationContextUtil implements ApplicationContextAware {

    private static final Logger LOGGER = LoggerFactory.getLogger(ApplicationContextUtil.class);

    /**
     * 应用上下文，静态是为了其他非Spring托管类使用
     */
    private static ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext context) throws BeansException {

        ApplicationContextUtil.context = context;
    }

    /**
     * 获取 bean 对象，非Spring 托管类使用
     * 
     * @param name
     * @return
     */
    public static Object getSpringBean(String name) {

        Object bean = null;
        try {
            bean = ApplicationContextUtil.context.getBean(name);
        }
        catch (Exception e) {
            LOGGER.warn("NoSuchBean:[{}]", name);
        }
        return bean;
    }

    /**
     * 获取 bean 对象，Spring 托管类使用
     * 
     * @param name
     * @return
     */
    public Object getBean(String name) {

        return getSpringBean(name);
    }
}
```