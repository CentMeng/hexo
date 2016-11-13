---
title: Webview支持https 双向认证(SSL)
date: 2016-10-22 21:11:35
categories:
- Android技术
tags:
- WebView
---


## HTTPS
首先我们在这之前先了解一下https
### Https相关知识
，HTTPS相当于HTTP的安全版本了，它在HTTP的之下加入了SSL (Secure Socket Layer)，这里说他安全就靠这个SSL了。SSL位于TCP/IP和HTTP协议之间，那么SSL有什么作用呢？

- 认证用户和服务器，确保数据发送到正确的客户机和服务器；(验证证书)
- 加密数据以防止数据中途被窃取；（加密）
- 维护数据的完整性，确保数据在传输过程中不被改变。（摘要算法）

### Https工作原理
HTTPS在传输数据之前需要客户端（浏览器）与服务端（网站）之间进行一次握手，在握手过程中将确立双方加密传输数据的密码信息。握手过程的简单描述如下：
1. 浏览器将自己支持的一套加密算法、HASH算法发送给网站。
2. 网站从中选出一组加密算法与HASH算法，并将自己的身份信息以证书的形式发回给浏览器。证书里面包含了网站地址，加密公钥，以及证书的颁发机构等信息。
3. 浏览器获得网站证书之后，开始验证证书的合法性，如果证书信任，则生成一串随机数字作为通讯过程中对称加密的秘钥。然后取出证书中的公钥，将这串数字以及HASH的结果进行加密，然后发给网站。
4. 网站接收浏览器发来的数据之后，通过私钥进行解密，然后HASH校验，如果一致，则使用浏览器发来的数字串使加密一段握手消息发给浏览器。
5. 浏览器解密，并HASH校验，没有问题，则握手结束。接下来的传输过程将由之前浏览器生成的随机密码并利用对称加密算法进行加密。

***握手过程中如果有任何错误，都会使加密连接断开，从而阻止了隐私信息的传输。***

以上流程协议，参考网上资料，不一定正确，如有错误希望指出。

根据上面的流程，我们可以看到服务器端会有一个证书，在交互过程中客户端需要去验证证书的合法性，对于权威机构颁发的证书当然我们会直接认为合法。对于自己造的证书，那么我们就需要去校验合法性了，也就是说我们只需要假如使用OkhttpClient去信任这个证书就可以畅通的进行通信了。

当然，对于自签名的网站的访问，网上的部分的做法是直接设置信任所有的证书，对于这种做法肯定是有风险的，所以这里我们不去介绍了，有需要自己去查。

## WebView支持https

OK，该说重点了，对于这方面，我们要从3个Android版本来写
- Android 4.0以下，即14以下，不包含14，如果最小支持14，则此处可忽略

```Java

 /**
     * 设置webview的ssl双向认证
     * 注意：该方法只支持android4.0（不包含）以下
     * 该方法调用一次即可
     * <P>Author : Vincent.M </P>
     */
    public boolean setWebViewSSLCert() {
        boolean issuc = false;// true 代表验证和设置成功
        if (Build.VERSION.SDK_INT >= 14) {
            return issuc;
        }

        InputStream[] inputStreams = new InputStream[1];
        try {
            inputStreams[0] = this.getAssets().open("key.cer");
        } catch (IOException e) {
            e.printStackTrace();
        }
        HttpsUtils.SSLParams sslParams = HttpsUtils.getSslSocketFactory(inputStreams, null, null);
        try {
            Field[] arrayOfField = Class.forName(
                    "android.net.http.HttpsConnection").getDeclaredFields();
            for (Field localField : arrayOfField) {
                if (localField.getName().equals("mSslSocketFactory")) {//采用反射的方式修改mSslSocketFactory变量
                    localField.setAccessible(true);
                    localField.set(null, sslParams.sSLSocketFactory);
                    issuc = true;
                    break;
                }
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return issuc;
    }
```

[HttpsUtils](https://github.com/CentMeng/baseFramework/blob/master/networkcore/src/main/java/com/msj/networkcore/https/HttpsUtils.java)方法

```Java
public class HttpsUtils
{
    public static class SSLParams
    {
        public SSLSocketFactory sSLSocketFactory;
        public X509TrustManager trustManager;
    }

    public static SSLParams getSslSocketFactory(InputStream[] certificates, InputStream bksFile, String password)
    {
        SSLParams sslParams = new SSLParams();
        try
        {
            TrustManager[] trustManagers = prepareTrustManager(certificates);
            KeyManager[] keyManagers = prepareKeyManager(bksFile, password);
            SSLContext sslContext = SSLContext.getInstance("TLS");
            X509TrustManager trustManager = null;
            if (trustManagers != null)
            {
                trustManager = new MyTrustManager(chooseTrustManager(trustManagers));
            } else
            {
                trustManager = new UnSafeTrustManager();
            }
            sslContext.init(keyManagers, new TrustManager[]{trustManager},null);
            sslParams.sSLSocketFactory = sslContext.getSocketFactory();
            sslParams.trustManager = trustManager;
            return sslParams;
        } catch (NoSuchAlgorithmException e)
        {
            throw new AssertionError(e);
        } catch (KeyManagementException e)
        {
            throw new AssertionError(e);
        } catch (KeyStoreException e)
        {
            throw new AssertionError(e);
        }
    }

    private class UnSafeHostnameVerifier implements HostnameVerifier
    {
        @Override
        public boolean verify(String hostname, SSLSession session)
        {
            return true;
        }
    }

    private static class UnSafeTrustManager implements X509TrustManager
    {
        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType)
                throws CertificateException
        {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType)
                throws CertificateException
        {
        }

        @Override
        public X509Certificate[] getAcceptedIssuers()
        {
            return new X509Certificate[]{};
        }
    }

    private static TrustManager[] prepareTrustManager(InputStream... certificates)
    {
        if (certificates == null || certificates.length <= 0) return null;
        try
        {

            CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
            KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
            keyStore.load(null);
            int index = 0;
            for (InputStream certificate : certificates)
            {
                String certificateAlias = Integer.toString(index++);
                keyStore.setCertificateEntry(certificateAlias, certificateFactory.generateCertificate(certificate));
                try
                {
                    if (certificate != null)
                        certificate.close();
                } catch (IOException e)

                {
                }
            }
            TrustManagerFactory trustManagerFactory = null;

            trustManagerFactory = TrustManagerFactory.
                    getInstance(TrustManagerFactory.getDefaultAlgorithm());
            trustManagerFactory.init(keyStore);

            TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();

            return trustManagers;
        } catch (NoSuchAlgorithmException e)
        {
            e.printStackTrace();
        } catch (CertificateException e)
        {
            e.printStackTrace();
        } catch (KeyStoreException e)
        {
            e.printStackTrace();
        } catch (Exception e)
        {
            e.printStackTrace();
        }
        return null;

    }

    private static KeyManager[] prepareKeyManager(InputStream bksFile, String password)
    {
        try
        {
            if (bksFile == null || password == null) return null;

            KeyStore clientKeyStore = KeyStore.getInstance("BKS");
            clientKeyStore.load(bksFile, password.toCharArray());
            KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
            keyManagerFactory.init(clientKeyStore, password.toCharArray());
            return keyManagerFactory.getKeyManagers();

        } catch (KeyStoreException e)
        {
            e.printStackTrace();
        } catch (NoSuchAlgorithmException e)
        {
            e.printStackTrace();
        } catch (UnrecoverableKeyException e)
        {
            e.printStackTrace();
        } catch (CertificateException e)
        {
            e.printStackTrace();
        } catch (IOException e)
        {
            e.printStackTrace();
        } catch (Exception e)
        {
            e.printStackTrace();
        }
        return null;
    }

    private static X509TrustManager chooseTrustManager(TrustManager[] trustManagers)
    {
        for (TrustManager trustManager : trustManagers)
        {
            if (trustManager instanceof X509TrustManager)
            {
                return (X509TrustManager) trustManager;
            }
        }
        return null;
    }


    private static class MyTrustManager implements X509TrustManager
    {
        private X509TrustManager defaultTrustManager;
        private X509TrustManager localTrustManager;

        public MyTrustManager(X509TrustManager localTrustManager) throws NoSuchAlgorithmException, KeyStoreException
        {
            TrustManagerFactory var4 = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
            var4.init((KeyStore) null);
            defaultTrustManager = chooseTrustManager(var4.getTrustManagers());
            this.localTrustManager = localTrustManager;
        }


        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException
        {

        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException
        {
            try
            {
                defaultTrustManager.checkServerTrusted(chain, authType);
            } catch (CertificateException ce)
            {
                localTrustManager.checkServerTrusted(chain, authType);
            }
        }


        @Override
        public X509Certificate[] getAcceptedIssuers()
        {
            return new X509Certificate[0];
        }
    }
}

```

- Android 4.0及以上版本，Android 5.0（不包含）以下,下面这段代码是通用的，在4.0和5.0，不同点在设置WebViewClient中

```
 private X509Certificate[] mX509Certificates;
    private PrivateKey mPrivateKey;

    private void initPrivateKeyAndX509Certificate()
            throws Exception {
        // 创建一个证书库，并将证书导入证书库
        try {
            InputStream input = this.getContext().getResources().openRawResource(R.raw.keystore);
            KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
            keyStore.load(input, CERTFILE_PASSWORD.toCharArray());
            Enumeration<?> localEnumeration;
            localEnumeration = keyStore.aliases();
            while (localEnumeration.hasMoreElements()) {
                String str3 = (String) localEnumeration.nextElement();
                mPrivateKey = (PrivateKey) keyStore.getKey(str3,
                        CERTFILE_PASSWORD.toCharArray());
                if (mPrivateKey == null) {
                    continue;
                } else {
                    Certificate[] arrayOfCertificate = keyStore
                            .getCertificateChain(str3);
                    mX509Certificates = new X509Certificate[arrayOfCertificate.length];
                    for (int j = 0; j < mX509Certificates.length; j++) {
                        mX509Certificates[j] = ((X509Certificate) arrayOfCertificate[j]);
                    }
                }
            }
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (CertificateException e) {
            e.printStackTrace();
        } catch (KeyStoreException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    
```

***这里在WebViewClient来区分4.0和5.0的不同点***

```Java
webview.setWebViewClient(new WebViewClient() {
            

            //支持处理https请求，默认是handler.cancel();
            @Override
            public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
                handler.proceed();
            }

               //5.0的处理            @TargetApi(Build.VERSION_CODES.LOLLIPOP)
            @Override
            public void onReceivedClientCertRequest(WebView view, ClientCertRequest request) {
                super.onReceivedClientCertRequest(view, request);
                LogUtils.e("msj", "onReceivedClientCertRequest");
                request.proceed(mPrivateKey, mX509Certificates);
            }

//4.0.3,4.1.2的处理 4.2.2,4.3.1使用类 WebViewClientClassicExt 
 public void onReceivedClientCertRequest(WebView view,
                                                    ClientCertRequestHandler handler, String host_and_port) {
                //注意该方法是调用的隐藏函数接口。这儿是整个验证的技术难点：就是如何调用隐藏类的接口。
                    //方法：去下载一个android4.2版本全编译后的class.jar 然后导入到工程中
                if(((mX509Certificates != null) && (mX509Certificates.length !=0))){
                    handler.proceed(mPrivateKey, mX509Certificates);
                }else{
                    handler.cancel();
                }
            }

```

***再书写4.0的时候会发现，有报错,有两种解决方式***

- 到android 4.2 源码环境下编译
- 去下载一个全编译的[class.jar](http://download.csdn.net/detail/u011930471/9660228)
