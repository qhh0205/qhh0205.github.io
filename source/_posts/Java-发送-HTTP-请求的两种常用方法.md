---
title: Java 发送 HTTP 请求的两种常用方法
date: 2021-05-16 16:02:36
categories: Java
tags: Java
---

本文主要介绍在 Java 编程中发送 HTTP 请求的两种常用方法：

 - JDK 原生的 HttpURLConnection 发送 HTTP 请求
 - [Apache HhttpClient](http://hc.apache.org/httpcomponents-client-5.1.x/) 发送 HTTP 请求
 
两种方法都可以发送 HTTP 请求，第一种是 Java 原生的，因此使用起来相对麻烦一些，第二种是通过第三方的包来实现，这个包是 Apache 旗下的专门用来发送 HTTP 请求的 HttpClient 包，是对 Java 原生的 HttpURLConnection 扩展，因此功能也更加强大，使用起来也相对简单一些，目前这种方式发送 HTTP 请求应用比较广泛，因此主要学习这种方式。

Apache HttpClient 官方文档见这里：[http://hc.apache.org/httpcomponents-client-5.1.x/](http://hc.apache.org/httpcomponents-client-5.1.x/)

Talk is cheap. Show me the code. 下面看具体使用代码示例。

#### 1、JDK 原生的 HttpURLConnection 发送 HTTP 请求 GET/POST
基于 JDK 原生的 HttpURLConnection 类，简单封装了下 HTTP GET 和 POST 请求方法，重在学习使用。
```java
package com.example.demo.common.utils;
/**
 * Created by qianghaohao on 2021/5/16
 */

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;

/**
 * @description: 使用 java 原生 HttpURLConnection 类发送 http 请求
 * @author: qianghaohao
 * @time: 2021/5/16
 */
public class HttpURLConnectionUtils {
    public static String doGet(String httpUrl) {
        HttpURLConnection connection = null;
        InputStream inputStream = null;
        BufferedReader bufferedReader = null;
        String result = null;
        try {
            // 创建远程url连接对象
            URL url = new URL(httpUrl);

            // 通过远程url连接对象打开一个连接，强转成httpURLConnection类
            connection = (HttpURLConnection) url.openConnection();

            // 设置连接方式：get
            connection.setRequestMethod("GET");

            // 设置连接主机服务器的超时时间：15000毫秒
            connection.setConnectTimeout(15000);

            // 设置读取远程返回的数据时间：60000毫秒
            connection.setReadTimeout(60000);

            // 通过connection连接，获取输入流
            if (connection.getResponseCode() == 200) {
                inputStream = connection.getInputStream();
                // 封装输入流is，并指定字符集
                bufferedReader = new BufferedReader(new InputStreamReader(inputStream, "UTF-8"));
                // 存放数据
                StringBuilder sb = new StringBuilder();
                String temp;
                while ((temp = bufferedReader.readLine()) != null) {
                    sb.append(temp);
                    sb.append(System.lineSeparator());  // 这里需要追加换行符，默认读取的流没有换行符，需要加上才能符合预期
                }
                result = sb.toString();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 关闭资源
            if (null != bufferedReader) {
                try {
                    bufferedReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (null != inputStream) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            if (null != connection) {
                connection.disconnect();// 关闭远程连接
            }
        }
        return result;
    }


    public static String doPost(String httpUrl, String param) {
        HttpURLConnection connection = null;
        InputStream is = null;
        OutputStream os = null;
        BufferedReader br = null;
        String result = null;
        try {
            URL url = new URL(httpUrl);
            // 通过远程url连接对象打开连接
            connection = (HttpURLConnection) url.openConnection();

            // 设置连接请求方式
            connection.setRequestMethod("POST");

            // 设置连接主机服务器超时时间：15000毫秒
            connection.setConnectTimeout(15000);

            // 设置读取主机服务器返回数据超时时间：60000毫秒
            connection.setReadTimeout(60000);

            // 默认值为：false，当向远程服务器传送数据/写数据时，需要设置为true
            connection.setDoOutput(true);

            // 设置传入参数的格式:请求参数应该是 name1=value1&name2=value2 的形式。
            connection.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");

            // 通过连接对象获取一个输出流
            os = connection.getOutputStream();

            // 通过输出流对象将参数写出去/传输出去,它是通过字节数组写出的
            os.write(param.getBytes());

            // 通过连接对象获取一个输入流，向远程读取
            if (connection.getResponseCode() == 200) {
                is = connection.getInputStream();
                // 对输入流对象进行包装:charset根据工作项目组的要求来设置
                br = new BufferedReader(new InputStreamReader(is, "UTF-8"));
                StringBuilder sb = new StringBuilder();
                String temp;
                // 循环遍历一行一行读取数据
                while ((temp = br.readLine()) != null) {
                    sb.append(temp);
                    sb.append(System.lineSeparator());
                }
                result = sb.toString();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 关闭资源
            if (null != br) {
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (null != os) {
                try {
                    os.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (null != is) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (null != connection) {
                // 断开与远程地址url的连接
                connection.disconnect();
            }
        }
        return result;
    }
}
```

#### 2、[Apache HhttpClient](http://hc.apache.org/httpcomponents-client-5.1.x/) 发送 HTTP 请求 GET/POST
基于 [Apache HhttpClient](http://hc.apache.org/httpcomponents-client-5.1.x/)，简单封装了下 HTTP GET 和 POST 请求方法，重在学习使用。
```java
package com.example.demo.common.utils;
/**
 * Created by qianghaohao on 2021/5/16
 */

import org.apache.http.HttpEntity;
import org.apache.http.NameValuePair;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * @description: 使用 Apache HttpClient 发送 http 请求
 * @author: qianghaohao
 * @time: 2021/5/16
 */
public class HttpClientUtils {
    public static String doGet(String url) {
        String content = null;
        // 创建 HttpClient 对象
        CloseableHttpClient httpclient = HttpClients.createDefault();

        // 创建 Http GET 请求
        HttpGet httpGet = new HttpGet(url);

        CloseableHttpResponse response = null;
        try {
            // 执行请求
            response = httpclient.execute(httpGet);
            // 判断返回状态是否为200
            if (response.getStatusLine().getStatusCode() == 200) {
                //响应体内容
                content = EntityUtils.toString(response.getEntity(), "UTF-8");
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (response != null) {
                try {
                    response.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            try {
                httpclient.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        return content;
    }

    public static String doPost(String url, Map<String, Object> paramMap) {
        CloseableHttpClient httpClient = null;
        CloseableHttpResponse httpResponse = null;
        String result = null;
        // 创建httpClient实例
        httpClient = HttpClients.createDefault();

        // 创建httpPost远程连接实例
        HttpPost httpPost = new HttpPost(url);

        // 设置请求头
        httpPost.addHeader("Content-Type", "application/x-www-form-urlencoded");

        // 封装post请求参数
        if (null != paramMap && paramMap.size() > 0) {
            List<NameValuePair> nvps = new ArrayList<NameValuePair>();
            // 通过map集成entrySet方法获取entity
            Set<Map.Entry<String, Object>> entrySet = paramMap.entrySet();
            // 循环遍历，获取迭代器
            for (Map.Entry<String, Object> mapEntry : entrySet) {
                nvps.add(new BasicNameValuePair(mapEntry.getKey(), mapEntry.getValue().toString()));
            }
            // 为httpPost设置封装好的请求参数
            try {
                httpPost.setEntity(new UrlEncodedFormEntity(nvps, "UTF-8"));
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        }

        try {
            // httpClient对象执行post请求,并返回响应参数对象
            httpResponse = httpClient.execute(httpPost);
            // 从响应对象中获取响应内容
            HttpEntity entity = httpResponse.getEntity();
            result = EntityUtils.toString(entity);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 关闭资源
            if (null != httpResponse) {
                try {
                    httpResponse.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (null != httpClient) {
                try {
                    httpClient.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        return result;
    }
}
```

#### 3、上面封装类使用示例
```java
    @Test
    public void httpDoGetTest() {
        // Java 原生 HttpURLConnection 发送 HTTP 请求测试
        String url = "https://httpbin.org/get";
        String result = HttpURLConnectionUtils.doGet(url);
        System.out.println(result);

        // Apache HttpClient 发送 http 请求测试
        System.out.println("==============");
        result = HttpClientUtils.doGet(url);
        System.out.println(result);
    }
    
    @Test
    public void httpDoPostTest() {
        // Java 原生 HttpURLConnection 发送 HTTP 请求测试
        String url = "https://httpbin.org/post";
        String urlParameters = "name=Jack&occupation=programmer";
        String result = HttpURLConnectionUtils.doPost(url, urlParameters);
        System.out.println(result);

        // Apache HttpClient 发送 http 请求测试
        System.out.println("==============");
        Map<String, Object> params = new HashMap<>();
        params.put("name", "Jack");
        params.put("occupation", "programmer");
        result = HttpClientUtils.doPost(url, params);
        System.out.println(result);
    }
```

