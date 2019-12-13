---
title: 记一次ftp并发导致的bug
date: 2018/4/20
categories:
  - java
tags:
  - 并发
comments: true
abbrlink: 49197
img:
---
ThreadLocal 适用于如下两种场景

- 每个线程需要有自己单独的实例

- 实例需要在多个方法中共享，但不希望被多线程共享



```

	private ThreadLocal<FTPClient> ftpClientThreadLocal = new ThreadLocal<FTPClient>();

	private FTPClient getFTPClient()  {
		if (ftpClientThreadLocal.get() != null && ftpClientThreadLocal.get().isConnected()) {
			return ftpClientThreadLocal.get();
		}else{
			FTPClient ftpClient = new FTPClient(); //构造一个FtpClient实例

			ftpClient.setControlEncoding("UTF-8");
//			ftpClient.completePendingCommand();

			ftpClient.enterLocalPassiveMode();
//			ftpClient.enterLocalActiveMode();
			ftpClient.setBufferSize(1024*2);


			loginFtp(ftpClient);

			try {
				ftpClient.setFileType(FTPClient.BINARY_FILE_TYPE);
			} catch (IOException e) {
				e.printStackTrace();
			}

			ftpClientThreadLocal.set(ftpClient);
			return ftpClient;
		}
	}

```


```
FTPClient ftpClient = getFTPClient();

```

