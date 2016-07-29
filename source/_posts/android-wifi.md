---
title: 安卓WiFi传输相关技术 
date: 2016-07-29 18:10:38
tags: [Android, Wi-Fi]
---

## 大纲
1. Wifi传输概述
2. `WifiP2pManager`类的介绍和使用
3. Wifi热点创建相关技术
4. 基于Wifi的文件传输方法
5. Demo的展示与介绍
6. 小结

<!--more-->

## 1. Wifi传输概述

### 1.1 步骤

1. 发现对方IP地址：涉及`WifiP2pManager`和Wifi热点的创建
2. 监听端口，建立Socket通讯
3. 传输文件或其他信息

### 1.2 应用
1. 应用、照片、视频等文件的分发和传输
2. 手机与手机，手机与PC网页的通讯、文件传输

## 2. `WifiP2pManager`类的介绍和使用
该[方法](http://developer.android.com/intl/zh-cn/reference/android/net/wifi/p2p/WifiP2pManager.html)提供了对**WiFi peer-to-peer**传输的管理API，从而使得应用可以发现可用的peer，设置到peer的连接以及查询peer列表。本质上其功能也可以通过UDP广播实现。

### 2.1 简介
WiFi p2p常用方法

~~~java
// Initialization 
public Channel initialize(Context srcContext, Looper srcLooper, ChannelListener listener);
public void connect(Channel c, WifiP2pConfig config, ActionListener listener);
public void discoverPeers(Channel c, ActionListener listener);
public void createGroup(Channel c, ActionListener listener);

// Request Info
public void requestPeers(Channel c, PeerListListener listener);
public void requestConnectionInfo(Channel c, ConnectionInfoListener listener);
public void requestGroupInfo(Channel c, GroupInfoListener listener);

// ...
~~~

WiFi p2p常用监听器

~~~java
public interface ActionListener {
    public void onSuccess();
    public void onFailure(int reason);
}
public interface PeerListListener {
    public void onPeersAvailable(WifiP2pDeviceList peers);
}
public interface ChannelListener {
    public void onChannelDisconnected();
}
public interface ConnectionInfoListener {
    public void onConnectionInfoAvailable(WifiP2pInfo info);
}
public interface GroupInfoListener {
    public void onGroupInfoAvailable(WifiP2pGroup group);
}
public interface ServiceResponseListener {
    public void onServiceAvailable(int protocolType,
            byte[] responseData, WifiP2pDevice srcDevice);
}
// ...
~~~   

WiFi p2p Intent

~~~java
@SdkConstant(SdkConstantType.BROADCAST_INTENT_ACTION)
public static final String WIFI_P2P_CONNECTION_CHANGED_ACTION;
public static final String WIFI_P2P_PEERS_CHANGED_ACTION;
public static final String WIFI_P2P_STATE_CHANGED_ACTION;
public static final String WIFI_P2P_THIS_DEVICE_CHANGED_ACTION;
~~~

### 2.2 初始化

#### 2.2.1 对wifi p2p intent广播进行监听

~~~java
public class WiFiDirectBroadcastReceiver extends BroadcastReceiver {

    private WifiP2pManager mManager;
    private Channel mChannel;
    private MyWiFiActivity mActivity;

    public WiFiDirectBroadcastReceiver(WifiP2pManager manager, Channel channel,
            MyWifiActivity activity) {
        super();
        this.mManager = manager;
        this.mChannel = channel;
        this.mActivity = activity;
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();

        if (WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION.equals(action)) {
            // Check to see if Wi-Fi is enabled and notify appropriate activity
        } else if (WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION.equals(action)) {
            // Call WifiP2pManager.requestPeers() to get a list of current peers
        } else if (WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION.equals(action)) {
            // Respond to new connection or disconnections
        } else if (WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION.equals(action)) {
            // Respond to this device's wifi state changing
        }
    }
}
~~~

#### 2.2.2 创建wifi p2p应用
1. 申请权限

~~~xml
<uses-sdk android:minSdkVersion="14" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
~~~

2. 初始化`WiFiP2PManager`，设置`IntentFilter`并注册广播接收对象

~~~java
IntentFilter mIntentFilter;
@Override
protected void onCreate(Bundle savedInstanceState){
    mManager = (WifiP2pManager) getSystemService(Context.WIFI_P2P_SERVICE);
    mChannel = mManager.initialize(this, getMainLooper(), null);
    mReceiver = new WiFiDirectBroadcastReceiver(mManager, mChannel, this);
    
    mIntentFilter = new IntentFilter();
    mIntentFilter.addAction(WifiP2pManager.WIFI_P2P_STATE_CHANGED_ACTION);
    mIntentFilter.addAction(WifiP2pManager.WIFI_P2P_PEERS_CHANGED_ACTION);
    mIntentFilter.addAction(WifiP2pManager.WIFI_P2P_CONNECTION_CHANGED_ACTION);
    mIntentFilter.addAction(WifiP2pManager.WIFI_P2P_THIS_DEVICE_CHANGED_ACTION);
}
@Override
protected void onResume() {
    super.onResume();
    registerReceiver(mReceiver, mIntentFilter);
}
@Override
protected void onPause() {
    super.onPause();
    unregisterReceiver(mReceiver);
}
~~~

### 2.3 发现方法

~~~java
// Descover methods
public void discoverPeers(Channel c, ActionListener listener);
public void discoverServices(Channel c, ActionListener listener)；

// Listeners
public interface ActionListener {
        public void onSuccess();
        public void onFailure(int reason);
}

~~~

## 3. Wifi热点创建相关技术
安卓系统将热点控制API设置为`@hide`属性，需要使用反射机制调用。Wifi热点的创建相关技术主要分为两步：

1. 创建`WifiConfiguration`， 调用`setWifiApEnabled`函数讲配置文件写入系统，从而开启Wifi热点。

	~~~java
	private void createWifiAp(String ssid, String password) {
	        if (mWifiManager.isWifiEnabled()) {
	            mWifiManager.setWifiEnabled(false);
	        }
	
	        mNetConfig = new WifiConfiguration();
	
	        mNetConfig.SSID = ssid;
	        mNetConfig.preSharedKey = password;
	        mNetConfig.status = WifiConfiguration.Status.ENABLED;
	        mNetConfig.allowedAuthAlgorithms.set(WifiConfiguration.AuthAlgorithm.OPEN);
	        mNetConfig.allowedProtocols.set(WifiConfiguration.Protocol.RSN);
	        mNetConfig.allowedProtocols.set(WifiConfiguration.Protocol.WPA);
	        mNetConfig.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.TKIP);
	        mNetConfig.allowedGroupCiphers.set(WifiConfiguration.GroupCipher.CCMP);
	        mNetConfig.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.TKIP);
	        mNetConfig.allowedPairwiseCiphers.set(WifiConfiguration.PairwiseCipher.CCMP);
	        mNetConfig.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.WPA_PSK);//NONE);
	
	        try {
	            Method setWifiApMethod = mWifiManager.getClass().getMethod("setWifiApEnabled", WifiConfiguration.class, boolean.class);
	            boolean apstatus = (Boolean) setWifiApMethod.invoke(mWifiManager, mNetConfig, true);
	
	            Method isWifiApEnabledmethod = mWifiManager.getClass().getMethod("isWifiApEnabled");
	            while (!(Boolean) isWifiApEnabledmethod.invoke(mWifiManager)) {
	            }
	            ;
	            Method getWifiApStateMethod = mWifiManager.getClass().getMethod("getWifiApState");
	            int apstate = (Integer) getWifiApStateMethod.invoke(mWifiManager);
	            Method getWifiApConfigurationMethod = mWifiManager.getClass().getMethod("getWifiApConfiguration");
	            mNetConfig = (WifiConfiguration) getWifiApConfigurationMethod.invoke(mWifiManager);
	            Log.d("CLIENT", "\nSSID:" + mNetConfig.SSID + "\nPassword:" + mNetConfig.preSharedKey + "\n");
	
	        } catch (Exception e) {
	            Log.e(TAG, e.toString());
	            e.printStackTrace();
	        }
	    }
	~~~

2. 下载端连接后，扫描`/proc/net/arp`文件获取连接到主机的IP。当然也可以等下载机访问我们指定的主机网址，再通过请求知道下载机的IP地址，这里暂不累述。

   ~~~java
   /**
	 * Gets a list of the clients connected to the Hotspot 
	 * @param onlyReachables {@code false} if the list should contain unreachable (probably disconnected) clients, {@code true} otherwise
	 * @param reachableTimeout Reachable Timout in miliseconds
	 * @param finishListener, Interface called when the scan method finishes 
	 */
	public void getClientList(final boolean onlyReachables, final int reachableTimeout, final FinishScanListener finishListener) {


		Runnable runnable = new Runnable() {
			public void run() {

				BufferedReader br = null;
				final ArrayList<ClientScanResult> result = new ArrayList<ClientScanResult>();
				
				try {
					br = new BufferedReader(new FileReader("/proc/net/arp"));
					String line;
					while ((line = br.readLine()) != null) {
						String[] splitted = line.split(" +");

						if ((splitted != null) && (splitted.length >= 4)) {
							// Basic sanity check
							String mac = splitted[3];

							if (mac.matches("..:..:..:..:..:..")) {
								boolean isReachable = InetAddress.getByName(splitted[0]).isReachable(reachableTimeout);

								if (!onlyReachables || isReachable) {
									result.add(new ClientScanResult(splitted[0], splitted[3], splitted[5], isReachable));
								}
							}
						}
					}
				} catch (Exception e) {
					Log.e(this.getClass().toString(), e.toString());
				} finally {
					try {
						br.close();
					} catch (IOException e) {
						Log.e(this.getClass().toString(), e.getMessage());
					}
				}

				// Get a handler that can be used to post to the main thread
				Handler mainHandler = new Handler(context.getMainLooper());
				Runnable myRunnable = new Runnable() {
					@Override
					public void run() {
						finishListener.onFinishScan(result);
					}
				};
				mainHandler.post(myRunnable);
			}
		};

		Thread mythread = new Thread(runnable);
		mythread.start();
	}
   ~~~

## 4. 基于Wifi的文件传输方法
基于WiFi的数据传输主要有两个技术点：一方面是文件传输，另一方面是下载端的网页实时获得文件传输的进度。前者可以直接使用`ServerSocket`实现，后者对于双方均安装我们的下载app的情况没有难度，但对于一边是app发送文件，一边是网页打开连接下载文件，就需要基于**`JavaScript`**的`XMLHttpRequest `可以实现主机和下载机网页的信息交流。

### 4.1 使用`ServerSocket`传输数据
~~~java
/**
 * Sends the HTTP response to the client, including headers (as applicable)
 * and content.
 */
private void processRequest(Socket client) throws IllegalStateException, IOException {
    Log.e(TAG, "Starting to process request");
    client.setKeepAlive(true);
    InputStream is = client.getInputStream();
    final int bufsize = 8192;
    byte[] buf = new byte[bufsize];
    int splitbyte = 0;
    int rlen = 0;
    {
        int read = is.read(buf, 0, bufsize);
        while (read > 0) {
            rlen += read;
            splitbyte = findHeaderEnd(buf, rlen);
            if (splitbyte > 0)
                break;
            read = is.read(buf, rlen, bufsize - rlen);
        }
    }

    // Create a BufferedReader for parsing the header.
    ByteArrayInputStream hbis = new ByteArrayInputStream(buf, 0, rlen);
    BufferedReader hin = new BufferedReader(new InputStreamReader(hbis));
    Properties pre = new Properties();
    Properties parms = new Properties();
    Properties header = new Properties();

    try {
        decodeHeader(hin, pre, parms, header);
    } catch (InterruptedException e1) {
        Log.e(TAG, "Exception: " + e1.getMessage());
        e1.printStackTrace();
    }
  
    String range = header.getProperty("range");
    cbSkip = 0;
    if (range != null) {
        Log.e(TAG, "range is: " + range);
        range = range.substring(6);
        int charPos = range.indexOf('-');
        if (charPos > 0) {
            range = range.substring(0, charPos);
        }
        cbSkip = Long.parseLong(range);
        Log.i(TAG, "range found!! " + cbSkip);
    }

    sendHTMLtext(HTMLManager.getDownloadingHTML(), client.getOutputStream());
    mSendState = SEND_STATE_SENDING;
}

private void sendHTMLtext(String htmlStr, OutputStream outputStream) {
    PrintWriter printWriter = new PrintWriter(outputStream);
    printWriter.println(htmlStr);
    printWriter.close();
}
~~~
### 4.2 使用`XMLHttpRequest`在网页上实时获取下载进度

~~~JavaScript
var xmlhttp = new XMLHttpRequest;
var url = "http://192.168.43.1:8080";
var timer = setInterval("loadurl(url)", 500); // 定时请求刷新

function loadurl(url) {
        if (xmlhttp != null) {
            xmlhttp.onreadystatechange = state_Change;
            xmlhttp.open("GET", url, true);
            xmlhttp.send(null);
        } else {
            alert("Your browser does not support XMLHTTP.");
        }
    }

function state_Change() {
    var infoBar = document.getElementsByClassName("cube")[0];
    if (xmlhttp.responseText == "") return;
    var percent = parseInt(xmlhttp.responseText.substring(0, 3));
    if (percent != NaN) {
        infoBar.innerHTML = xmlhttp.responseText; // 刷新DOM元素
        if (percent == 100) {
            clearInterval(timer);
            infoBar.innerHTML = "Complete. Please install it.";
        }
    }
}
~~~


## 5. Demo的展示与介绍
wifi热点的创建和文件传输
## 6. 小结
### 6.1 基于安卓的wifi传输 ==能做到什么==？
1. 局域网内，安装我们APP的手机之间的互相发现和连接
2. 热点的创建和定制
3. 免流量的高速、稳定的数据传输，相对于蓝牙和NFC具有很大优势
4. 一方安装了APP，另一方仍可通过热点连接，基于网页JS实现与主机APP的信息交互

### 6.2 基于安卓的wifi传输 ==不能做到什么==？==有哪些坑==？
1. 没有底层权限，很多服务器常见的设置都无法完成，比如无法设置`IPTable`
2. 安卓网页的局限，浏览器对JS的支持性和PC端不同，需要额外设计
3. 部分安卓厂商自己修改了官方的热点API，比如HTC，这使得在这些手机上创建热点需要额外配置




