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