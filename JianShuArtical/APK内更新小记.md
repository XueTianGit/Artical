##APK内更新APK

>简单流程

1. 请求服务器是否有版本更新
2. 下载新版APK
3. 安装APK

对于第一步这是OK的，下面来看一下第二步的实现思路和第三步的一些坑。

- APK的下载

启动Service，利用DownloadManager下载APK，并注册下载完成监听

>这里这个`service`不要指定多进程 `process`属性
    
```
DownloadManager manager = 
                    (DownloadManager) getSystemService(Context.DOWNLOAD_SERVICE);

DownloadManager.Request request = new DownloadManager.Request(Uri.parse(realUrl));
request.setVisibleInDownloadsUi(true);  
        request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE |DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED);

request.setMimeType("application/vnd.android.package-archive");
File file = new File(getPath(url));
request.setDestinationUri(Uri.fromFile(file));
downloadId = manager.enqueue(request); //下载任务的ID,用来校验任务
//注册任务下载完成，广播接收者
registerReceiver(mReceiver, 
                    new IntentFilter(DownloadManager.ACTION_DOWNLOAD_COMPLETE));

```
 
- APK安装

>apk安装一般需要进行文件MD5的校验，校验通过之后安装APK。

对于APK的安装应注意以下点：

申请安装权限：

`<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES" />`

在安装APK时，对于Android N (7.0)以上：
`系统修改了安全机制: 限定应用在默认情况下只能访问自身应用数据。所以当我们想通过File对象访问其它package数据时，就需要借助于ContentProvider、FileProvider这些组件，否则会报 FileUriExposedException 异常`

首先提供 FileProvider:

```
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="应用包名.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths"/>
</provider>
<paths>
    <external-path
        name="download"
        path="" />
        
    <external-files-path
        name="Download"
        path="" />
</paths>
```

对于不同Android版本，走不同的处理逻辑：

```
Intent install = new Intent(Intent.ACTION_VIEW);
install.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
File apkFile = new File(Environment.getExternalStorageDirectory() + "/download/" + "app.apk";

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    install.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
    Uri contentUri = FileProvider.getUriForFile(context, "应用报名.fileProvider", apkFile);
    install.setDataAndType(contentUri, "application/vnd.android.package-archive");
} else {
    install.setDataAndType(Uri.fromFile(apkFile), "application/vnd.android.package-archive");
}

startActivity(install);
```












