---
title: HttpUtil工具类
date: 2017-01-07 24:55:44
updated: 
categores:
permalink:
tags: [HttpUtil,工具类]
---
HttpUtil工具类使用HttpClients

```
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;
import java.security.GeneralSecurityException;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSession;
import org.apache.commons.io.IOUtils;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.NameValuePair;
import org.apache.http.client.config.RequestConfig;
import  org.apache.http.client.entity.UrlEncodedFormEntity;
import  org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.HttpClientUtils;
import  org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.conn.ssl.TrustStrategy;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import  org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.ssl.SSLContextBuilder;
import org.apache.http.util.EntityUtils;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.yeahka.loan.yunxin.constant.YunXinConstant;
import com.yeahka.util.JsonUtil;
public class YunXinHttpUtil {
     private static final Log log =  LogFactory.getLog(YunXinHttpUtil.class);
     private static PoolingHttpClientConnectionManager  connMgr;
     private static RequestConfig requestConfig;
     private static final int MAX_TIMEOUT = 30000;
     static {
          // 设置连接池
          connMgr = new  PoolingHttpClientConnectionManager();
          // 设置连接池大小
          connMgr.setMaxTotal(100);
          connMgr.setDefaultMaxPerRoute(connMgr.getMaxTotal());
          RequestConfig.Builder configBuilder =  RequestConfig.custom();
          // 设置连接超时
          configBuilder.setConnectTimeout(MAX_TIMEOUT);
          // 设置读取超时
          configBuilder.setSocketTimeout(MAX_TIMEOUT);
          // 设置从连接池获取连接实例的超时
          configBuilder.setConnectionRequestTimeout(MAX_TIMEOUT);
          // 在提交请求之前 测试连接是否可用
          //  configBuilder.setValidateAfterInactivity(true);
          requestConfig = configBuilder.build();
     }
     /**
      * 发送 GET 请求（HTTP/HTTPS），K-V形式
      *
      * @param url
      * @param params
      * @return
      */
     public static String doGet(String url, Map<String,  Object> params) {
          String apiUrl = url;
          CloseableHttpClient httpClient = null;
          if(url.startsWith("https:")){
              httpClient = HttpClients.custom()
                        .setSSLSocketFactory(createSSLConnSocketFactory())
                        .setConnectionManager(connMgr)
                        .setDefaultRequestConfig(requestConfig).build();
          }else{
              httpClient = HttpClients.createDefault();
          }
          StringBuffer param = new StringBuffer();
          
          int i = 0;
          for (String key : params.keySet()) {
              if (i == 0)
                   param.append("?");
              else
                   param.append("&");
               param.append(key).append("=").append(params.get(key));
              i++;
          }
          apiUrl += param;
          String result = null;
          
          try {
              HttpGet httpPost = new HttpGet(apiUrl);
              HttpResponse response =  httpClient.execute(httpPost);
              int statusCode =  response.getStatusLine().getStatusCode();
              System.out.println("执行状态码 : " +  statusCode);
              HttpEntity entity = response.getEntity();
              if (entity != null) {
                   InputStream instream =  entity.getContent();
                   result = IOUtils.toString(instream,  "UTF-8");
              }
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              if (httpClient != null) {
                   HttpClientUtils.closeQuietly(httpClient);
              }
          }
          log.info("request to : {"+apiUrl+"} , param :  {"+param.toString()+"} , result : {"+result+"}");
          return result;
     }
     
     /**
      * 发送 POST 请求（HTTP/HTTPS），K-V形式
      *
      * @param url
      *            API接口URL
      * @param params
      *            参数map
      * @return
      */
     public static String doPost(String url, Map<String,  Object> params) {
          CloseableHttpClient httpClient = null;
          String httpStr = null;
          HttpPost httpPost = null;
          CloseableHttpResponse response = null;
          if(url.startsWith("https:")){
              httpClient = HttpClients.custom()
                        .setSSLSocketFactory(createSSLConnSocketFactory())
                        .setConnectionManager(connMgr)
                        .setDefaultRequestConfig(requestConfig).build();
              httpPost = new HttpPost(url);
          }else{
              httpClient = HttpClients.createDefault();
              httpPost = new HttpPost(url);
          }
          try {
              httpPost.setConfig(requestConfig);
              List<NameValuePair> pairList = new  ArrayList<NameValuePair>();
              for (Map.Entry<String, Object> entry :  params.entrySet()) {
                   NameValuePair pair = new  BasicNameValuePair(entry.getKey(),
                             entry.getValue().toString());
                   pairList.add(pair);
              }
              httpPost.setEntity(new  UrlEncodedFormEntity(pairList, Charset
                        .forName("UTF-8")));
              response = httpClient.execute(httpPost);
              System.out.println(response.toString());
              HttpEntity entity = response.getEntity();
              httpStr = EntityUtils.toString(entity,  "UTF-8");
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              if (response != null) {
                   try {
                        EntityUtils.consume(response.getEntity());
                   } catch (IOException e) {
                        e.printStackTrace();
                   }
              }
          }
          return httpStr;
     }
     /**
      * 发送 POST 请求（HTTP/HTTPS），JSON形式
      *
      * @param url
      * @param json
      *            json对象
      * @return
      */
     public static String doPost(String url,Object obj) {
          CloseableHttpClient httpClient = null;
          String httpStr = null;
          HttpPost httpPost = null;
          CloseableHttpResponse response = null;
          if(url.startsWith("https:")){
              httpClient = HttpClients.custom()
                        .setSSLSocketFactory(createSSLConnSocketFactory())
                        .setConnectionManager(connMgr)
                        .setDefaultRequestConfig(requestConfig).build();
              httpPost = new HttpPost(url);
          }else{
              httpClient = HttpClients.createDefault();
              httpPost = new HttpPost(url);
          }
          try {
              String json = JsonUtil.obj2json(obj);
              log.info(json);
              ObjectMapper mapper = new ObjectMapper();
              httpPost.setConfig(requestConfig);
              httpPost.setHeader("Content-Type",  "application/json");
              httpPost.setHeader("MerId",  YunXinConstant.merId);
              httpPost.setHeader("SecretKey",  YunXinConstant.secretKey);
               httpPost.setHeader("SignedMsg",getSignMsg(mapper.writeValueAsString(obj)));
              StringEntity stringEntity = new  StringEntity(json,"UTF-8");// 解决中文乱码问题
              stringEntity.setContentEncoding("UTF-8");
               stringEntity.setContentType("application/json");
              httpPost.setEntity(stringEntity);
              response = httpClient.execute(httpPost);
              HttpEntity entity = response.getEntity();
              httpStr = EntityUtils.toString(entity,  "UTF-8");
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              if (response != null) {
                   try {
                        EntityUtils.consume(response.getEntity());
                   } catch (IOException e) {
                        e.printStackTrace();
                   }
              }
          }
          return httpStr;
     }
     private static String getSignMsg(String msg){
          RsaSignUtils utl = new  RsaSignUtils(YunXinConstant.privateKeyPath,YunXinConstant.privateCertPassword);
          return utl.generate(msg);
     }
     /**
      * 创建SSL安全连接
      *
      * @return
      */
     private static SSLConnectionSocketFactory  createSSLConnSocketFactory() {
          SSLConnectionSocketFactory sslsf = null;
          try {
              SSLContext sslContext = new  SSLContextBuilder().loadTrustMaterial(
                        null, new TrustStrategy() {
                             public boolean  isTrusted(X509Certificate[] chain,
                                      String authType)  throws CertificateException {
                                  return true;
                             }
                        }).build();
              sslsf = new  SSLConnectionSocketFactory(sslContext,
                        new HostnameVerifier() {
                   public boolean verify(String hostname,
                             SSLSession session) {
                        // TODO Auto-generated method  stub
                        return false;
                   }
              });
          } catch (GeneralSecurityException e) {
              e.printStackTrace();
          }
          return sslsf;
     }
}

```