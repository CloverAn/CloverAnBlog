前景：

```
目前在idea开发环境中，经常使用动态的yml来实现多环境配置切换。
目前市面上未检索到能够正常使用的动态获取yml数据源的配置工具类。
```



使用场景：

```
在maven项目下，在主pom.xml文件中，有配置<profiles><profile> </profile></profiles>动态数据源的情况下。
在需要根据不同环境配置除数据源属性以外的、业务需要的属性和内容的情况下。
```



工具：

```
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.core.io.ClassPathResource;
import java.util.Properties;

/**
  * @Description: 动态获取yml多环境配置文件数据
  * @Author: CloverAn
  * @Date: 2021/5/17
  * @Email: CloverAn@aliyun.com
  */
public class YmlConfigTool {
    private static Properties ymlProperties = new Properties();

    static {
        //1:加载配置文件
        YamlPropertiesFactoryBean yamlPropertiesFactoryBean = new YamlPropertiesFactoryBean();
        // 2:将加载的配置文件交给 YamlPropertiesFactoryBean
        yamlPropertiesFactoryBean.setResources(new ClassPathResource("application.yml"));
        // 3：将yml转换成 key：val
        Properties properties = yamlPropertiesFactoryBean.getObject();
        // 4: 将Properties 通过构造方法交给我们写的工具类
        yamlPropertiesFactoryBean.setResources(new ClassPathResource("application.yml"), new ClassPathResource(new StringBuilder().append("application-").append(properties.getProperty("spring.profiles.active")).append(".yml").toString()));
        ymlProperties = yamlPropertiesFactoryBean.getObject();
    }

    public static Properties getYmlProperties() {
        return ymlProperties;
    }

    public static void setYmlProperties(Properties ymlProperties) {
       YmlConfigTool.ymlProperties = ymlProperties;
    }

    public static YmlConfigTool create(Properties properties) {
        return new YmlConfigTool(properties);
    }

    public YmlConfigTool(Properties properties) {
        setYmlProperties(properties);
    }

    public static String getStr(String key) {
        return ymlProperties.getProperty(key);
    }

    public static Integer getInt(String key) {
        return Integer.valueOf(ymlProperties.getProperty(key));
    }

    public static Boolean getBoolean(String key) {
        return Boolean.valueOf(ymlProperties.getProperty(key));
    }
}

```



使用：

```
import org.springframework.stereotype.Component;

/**
 * @description: yml构造工具类
 * @author: CloverAn
 * @Eamil: CloverAn@aliyun.com
 * @create: 2021-05-17 08:51
 */
@Component
public class YmlConfigToolConstants {
    public  interface  test{
        String TEST = YmlConfigTool.getStr("demo.test");
    }
}

```



其他可以操作yml文件的工具：

```
		<!-- yaml文件操作 -->
		<dependency>
			<groupId>org.jyaml</groupId>
			<artifactId>jyaml</artifactId>
			<version>1.3</version>
		</dependency>
		<dependency>
			<groupId>org.yaml</groupId>
			<artifactId>snakeyaml</artifactId>
			<version>1.19</version>
		</dependency>
		<!-- yaml文件操作 -->
```

