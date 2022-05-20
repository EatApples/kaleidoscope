# fastjson 版本

```java

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.serializer.SerializerFeature;

/**
 *
 * @author zhen zhang
 *
 */
public class JSONHelper {

    private JSONHelper() {

    }

    public static String toString(Object obj, boolean isEncodeJSONString) {

        if (null == obj)
            return null;

        String str;

        if (String.class.isAssignableFrom(obj.getClass())) {
            str = (String) obj;
        } else {
            str = JSON.toJSONString(obj, SerializerFeature.DisableCircularReferenceDetect);
        }

        if (isEncodeJSONString == true) {
            str = str.replace("\"", "\\\"");
        }

        return str;
    }

    public static String toString(Object obj) {

        return toString(obj, false);
    }

    public static <T> T toObject(String jsonString, Class<T> c) {

        return toObject(jsonString, c, true);
    }

    public static <T> T toObject(String jsonString, Class<T> c, boolean isOrderFields) {

        if (null == c || StringUtils.isEmpty(jsonString)) {
            return null;
        }

        return JSON.parseObject(jsonString, c, Feature.OrderedField);
    }

    public static <T> List<T> toObjectArray(String jsonString, Class<T> c) {

        if (null == c || StringUtils.isEmpty(jsonString)) {
            return Collections.emptyList();
        }

        return JSON.parseArray(jsonString, c);
    }

    public static Object convertJO2POJO(Object data) {

        Class<?> dCls = data.getClass();

        if (JSONObject.class.isAssignableFrom(dCls)) {

            Map<String, Object> m = new LinkedHashMap<String, Object>();

            JSONObject jod = (JSONObject) data;

            for (String key : jod.keySet()) {

                Object attr = jod.get(key);

                Object attrObj = convertJO2POJO(attr);

                m.put(key, attrObj);
            }

            return m;

        } else if (JSONArray.class.isAssignableFrom(dCls)) {

            List<Object> l = new ArrayList<Object>();

            JSONArray joa = (JSONArray) data;

            for (Object o : joa) {

                Object attrObj = convertJO2POJO(o);

                l.add(attrObj);
            }

            return l;

        }

        return data;
    }
}

```

# jackson 版本

```java

import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;

@Slf4j
public class JSONHelper {

    private static final ObjectMapper MAPPER = new ObjectMapper();

    private JSONHelper() {
    }

    /**
     * toString
     *
     * @param obj
     * @return
     */
    public static String toString(Object obj) {

        if (null == obj) {
            return null;
        }
        try {
            return MAPPER.writeValueAsString(obj);
        } catch (Exception e) {
            log.error("", e);
        }
        return obj.toString();
    }

    /**
     * toObject
     *
     * @param jsonString
     * @param c
     * @param <T>
     * @return
     */
    public static <T> T toObject(String jsonString, Class<T> c) {

        if (null == c || StringUtils.isEmpty(jsonString)) {
            return null;
        }
        try {
            return MAPPER.readValue(jsonString, c);
        } catch (Exception e) {
            log.error("", e);
        }
        return null;
    }

    /**
     * toObjectArray
     *
     * @param jsonString
     * @param c
     * @param <T>
     * @return
     */
    public static <T> List<T> toObjectArray(String jsonString, Class<T> c) {

        if (null == c || StringUtils.isEmpty(jsonString)) {
            return Collections.emptyList();
        }
        try {
            JavaType javaType = MAPPER.getTypeFactory().constructParametricType(List.class, c);
            return MAPPER.readValue(jsonString, javaType);
        } catch (Exception e) {
            log.error("", e);
        }
        return null;
    }
}
```
