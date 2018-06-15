---
title: 基于jackson的json和object相互转换的工具类
date: 2017-01-07 23:55:44
updated: 
categores:
permalink:
tags: [jsonUtil,工具类]
---

```
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;

import org.apache.commons.lang3.StringUtils;
import org.springframework.util.CollectionUtils;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

/**
* 基于jackson的json和object相互转换的工具类
* @author 
* @date 2017年6月21日 上午10:00:23
*/
public final class JsonUtil {
    /** 私有构造 单例 */
    private JsonUtil(){
        
    }
    
    private static ObjectMapper objectMapper = null;
    
    static
    {
        // 将objectMapper 设置为全局静态缓存，提高调用效率
        objectMapper = new ObjectMapper();
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
    }
    
    /** 供外部调用 ObjectMapper */
    public static ObjectMapper getObjectMapper()
    {
        return objectMapper;
    }
    
    /**
     * json串转对象
     * @Description
     * @param jsonStr
     * @param clazz
     * @return
     */
    public static <T> T json2obj(String jsonStr, Class<T> clazz)
    {
        if (StringUtils.isEmpty(jsonStr))
        {
            return null;
        }

        T t = null;

        try
        {
            t = objectMapper.readValue(jsonStr, clazz);
        }
        catch (IOException e)
        {

        }
        return t;
    }
    
    /**
     * json串转map对象,前提是被转换的json串value为多组k-v
     * {"zhangjiajie":{"price":"fixed","type":"mountain"},"jiuzhaigou":{"price":"fixed","type":"mountain"}}
     * @Description
     * @param jsonStr
     * @param clazz
     * @return Map
     */
    public static <T> Map<String, T> json2map(String jsonStr, Class<T> clazz)
    {
        if (StringUtils.isEmpty(jsonStr))
        {
            return null;
        }

        Map<String, Map<String, Object>> map = null;
        try
        {
            map = objectMapper.readValue(jsonStr,
                    new TypeReference<Map<String, T>>()
                    {
                    });
        }
        catch (IOException e)
        {

        }

        if (CollectionUtils.isEmpty(map))
        {
            return null;
        }

        Map<String, T> result = new HashMap<String, T>();
        for (Entry<String, Map<String, Object>> entry : map.entrySet())
        {
            result.put(entry.getKey(), map2pojo(entry.getValue(), clazz));
        }
        return result;
    }
    
    /**
     * json转list对象
     * @Description
     * @param jsonStr
     * @param clazz
     * @return List
     */
    public static <T> List<T> json2list(String jsonStr, Class<T> clazz)
    {
        if (StringUtils.isEmpty(jsonStr))
        {
            return null;
        }

        List<Map<String, Object>> list = null;
        try
        {
            list = objectMapper.readValue(jsonStr,
                    new TypeReference<List<T>>()
                    {
                    });
        }
        catch (IOException e)
        {
        }

        // 非空校验
        if (CollectionUtils.isEmpty(list))
        {
            return null;
        }

        List<T> result = new ArrayList<T>();
        for (Map<String, Object> map : list)
        {
            result.add(map2pojo(map, clazz));
        }
        return result;
    }
    
    /**
     * 获取json串的某个键对应的值
     * @Description
     * @param jsonSrc
     * @param jsonKey
     * @return
     */
    public static String getJsonValue(String jsonSrc, String jsonKey)
    {
        if (StringUtils.isEmpty(jsonSrc) || StringUtils.isEmpty(jsonKey))
        {
            return null;
        }
        JsonNode node = json2obj(jsonSrc, JsonNode.class);
        
        if(node == null)
        {
            return null;
        }

        // 获取jsonKey数据
        JsonNode dataNode = node.get(jsonKey);

        if (null == dataNode)
        {
            return null;
        }

        return dataNode.toString();
    }
    
    /**
     * 对象转json串,维持基本类型,空值返回null

     * 示例

     * {"name":"alex","age":20,"address":null,"score":[{"subject":"math","score":98},{"subject":"art","score":50}]}
     * @Description
     * @param obj
     * @return
     */
    public static String obj2json(Object obj)
    {
        if (obj == null)
        {
            return null;
        }
        String jsonResult = null;
        try
        {
            jsonResult = objectMapper.writeValueAsString(obj);
        }
        catch (JsonProcessingException e)
        {
    
        }
    
        return jsonResult;
    }

    /**
     * 一般对象转JsonNode
     * @Description
     * @param obj
     * @return
     */
    public static JsonNode obj2node(Object obj)
    {
        if (null == obj)
        {
            return null;
        }

        JsonNode node = null;

        try
        {
            node = objectMapper.readTree(obj2json(obj));
        }
        catch (IOException e)
        {
        }

        return node;
    }
    
    public static <T> T obj2T(Object obj, Class<T> clazz)
    {
        if (null == obj)
        {
            return null;
        }

        T t = null;

        try
        {
            t = objectMapper.readValue(obj2json(obj), clazz);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }

        return t;
    }
    
    /**
     * map convert to javaBean
     */
    private static <T> T map2pojo(@SuppressWarnings("rawtypes")Map map, Class<T> clazz)
    {
        return objectMapper.convertValue(map, clazz);
    }
    
}


```