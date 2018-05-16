---
title: Java SFTP上传csv文件
categories: Java
---


## csv文件生成

引入javacsv.jar包
写csv文件代码
``` java
public String writerCSV(){
	String filePath = "\\upload\\001.csv";
	try{
		CsvWriter csv = new CsvWriter(filePath,',',Charset.forName(GBK));
		String[] content = {"天气","地点","人物"};
		String[] content1 = {"晴","七猫酒馆","白狼"};
		csv.writeRecord(content);
		csv.writeRecord(content1);
		
		csv.close();
	}catch(IOException e){
		e.printStackTrace();
	}
	return filePath;
}
```
## SFTP上传

导入jsch.jar包,编写工具类

``` java
public class SFTPUtil{
	private String username;
	private String password;
	private String host;
	private int port;
	
	private Session session = null;
	private ChannelSftp channel = null;
	
	public SFTPUtil(String username,String password,String host,int port){
		this.username = username;
		this.password = password;
		this.host = host;
		this.port = port;
	}
	
	public void login(){
		JSch jsch = new JSch();
		try{
			//获取会话
			session = jsch.getSession(username,host,port);
			if(session==null){
				throw new JSchException("session is null");
			}
			session.setPassword(password);
			//设置第一次登陆提示，可选值（ask|yes|no）
			session.setConfig("StrictHostKeyChecking","no");
			//登陆超时时间
			session.connect(30000);
			//创建sftp通信通道
			channel = (ChannlSftp) session.openChannl("sftp");
			channel.connect(1000);
		}catch(JSchException e){
			e.printStackTrace();
			seeion.disconnect();
			channel.disconnect();
		}
	}
	
	public void upload(String directory,String fileName){
		try{
			//切换目录
			channel.cd(directory);
			File file = new File(fileName);
			channel.put(new FileInputStream(file),file.getName());
		}catch (sftpException e) {
			e.printStackTrace();
		}catch (FileNotFoundException e) {
			e.printStackTrace();
		}
	}
	
	public void logout(){
		if (channel != null) {
	        if (channel.isConnected()) {
	        	channel.disconnect();
	        }
	    }
	    if (session != null) {
	        if (session.isConnected()) {
	            session.disconnect();
	        }
	    }
	}
}
```

## 测试

``` java
public static void main(String[] args){
	SftpUtil sftp = new SftpUtil("test", "123", "192.168.0.1", 22);
	Stirng fileName = writerCSV();
	sftp.login();
	sftp.upload("TEST", fileName);
	sftp.logout();
}
```
