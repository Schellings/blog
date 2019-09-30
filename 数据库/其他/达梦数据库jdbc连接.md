---
title: 达梦数据库jdbc连接
date: 2018-09-01 16:56:24
tags: 数据库
---
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;

import javax.imageio.ImageIO;
public class BasicApp {
/* 定义 DM JDBC 驱动串 */
	String jdbcString = "dm.jdbc.driver.DmDriver";
/*
 * 定义 DM URL 连接串
 * 此处/CAS是指与下方CAS创建的同名模式，达梦数据库通过登录的用户名来区分使用模式。
 */
	String urlString = "jdbc:dm://localhost:5236/CAS";
/* 定义连接用户名 */
	String userName = "CAS";
/* 定义连接用户口令 */
	String password = "CAS@Password";
	static /* 定义sql语句 */
/* String sqlString ="create table yujin3(a int,b int,c int);"; */
	String sqlString1 = "insert into yujin3  values(123,14,1234);";
/* 定义连接对象 */
	static Connection conn = null;
/* private static String sqlString1; */


/* 加载 JDBC 驱动程序
 * @throws SQLException 异常 */
	public void loadJdbcDriver() throws SQLException
	{
		try {
			System.out.println( "Loading JDBC Driver..." );
/*
 * 加载 JDBC 驱动程序
 * DriverManager.registerDriver(new dm.jdbc.driver.DmDriver());
 */
			Class.forName( jdbcString );
		} catch ( ClassNotFoundException e ) {
			throw new SQLException( "Load JDBC Driver Error1: " + e.getMessage() );
		} catch ( Exception ex ) {
			throw new SQLException( "Load JDBC Driver Error : "
						+ ex.getMessage() );
		}
	}


	public void connect() throws SQLException
	{
		try {
			System.out.println( "Connecting to DM Server..." );
/* 连接 DM 数据库 */
			conn = DriverManager.getConnection( urlString, userName, password );
		} catch ( SQLException e ) {
			throw new SQLException( "Connect to DM Server Error : "
						+ e.getMessage() );
		}
	}


/* 关闭连接
 * @throws SQLException 异常 */
	public void disConnect() throws SQLException
	{
		try {
/* 关闭连接 */
			conn.close();
			System.out.println( "close" );
		} catch ( SQLException e ) {
			throw new SQLException( "close connection error : " + e.getMessage() );
		}
	}


	public static void main( String args[] )
	{
		try {
			BasicApp basicApp = new BasicApp();
			/* 加载驱动程序 */
			basicApp.loadJdbcDriver();
			basicApp.connect();

			PreparedStatement pstmt1 = conn.prepareStatement( sqlString1 );
			/*
			 * pstmt1.setInt(1,11);
			 * pstmt1.setInt(2, 12);
			 * pstmt1.setInt(3, 123);
			 */

			pstmt1.execute();
			/* 关闭语句 */
			pstmt1.close();


			System.out.println( "OK!" );
			basicApp.disConnect();
		} catch ( SQLException e ) {
			System.out.println( e.getMessage() );
		}
	}
}
```

达梦 jdbc jar存放于达梦数据库安装目录。