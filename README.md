The shiro application based on Web and JdbcRealm.
===================================
#### pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>name.luoyong.shiro</groupId>
	<artifactId>shiroIniWebJdbcRealm</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<name>shiroIniWeb</name>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<dependencies>

		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.2</version>
		</dependency>

		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-core</artifactId>
			<version>1.2.3</version>
		</dependency>

		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-web</artifactId>
			<version>1.2.3</version>
		</dependency>

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>3.0-alpha-1</version>
			<scope>provided</scope>
		</dependency>

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
			<scope>provided</scope>
		</dependency>

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jsp-api</artifactId>
			<version>2.0</version>
			<scope>provided</scope>
		</dependency>
		
		<dependency>
    		<groupId>mysql</groupId>
    		<artifactId>mysql-connector-java</artifactId>
    		<version>5.1.6</version>
    	</dependency>
		
		<dependency>
    		<groupId>com.alibaba</groupId>
    		<artifactId>druid</artifactId>
    		<version>1.0.7</version>
   		</dependency>
   		
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.eclipse.jetty</groupId>
				<artifactId>jetty-maven-plugin</artifactId>
				<version>9.2.1.v20140609</version>
				<configuration>
					<scanIntervalSeconds>3</scanIntervalSeconds>
					<webAppConfig>
						<contextPath>/</contextPath>
					</webAppConfig>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>
```

#### shiro.ini
```ini
[main]
dataSource=com.alibaba.druid.pool.DruidDataSource
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql://yourHost:3306/shiro
dataSource.username=***
dataSource.password=***

dataSource.initialSize=10
dataSource.minIdle=20
dataSource.maxActive=100

jdbcRealm=name.luoyong.shiro.shiroIniWebJdbcRealm.servlet.MyJdbcRealm
jdbcRealm.permissionsLookupEnabled=true
jdbcRealm.dataSource=$dataSource

securityManager.realms=$jdbcRealm

authc.loginUrl=/login


[urls]
/ds=anon
/login=anon
/logout=logout
/**=authc
```

#### web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/j2ee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_3_0.xsd">

	<listener>
		<listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
	</listener>

	<filter>
		<filter-name>shiroFilter</filter-name>
		<filter-class>org.apache.shiro.web.servlet.ShiroFilter</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>shiroFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

</web-app>
```

#### LoginServlet.java
```java
package name.luoyong.shiro.shiroIniWeb.servlet;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.IncorrectCredentialsException;
import org.apache.shiro.authc.UnknownAccountException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.subject.Subject;

@WebServlet(urlPatterns="/login")
public class LoginServlet extends HttpServlet {
	
	private static final String loginJspUrl = "/login.jsp";
	private static final String loginSuccessJspUrl = "/loginSuccess.jsp";
	
	@Override
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		request.getRequestDispatcher("/login.jsp").forward(request, response);
	}
	
	@Override
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		
		Subject subject = SecurityUtils.getSubject();
		
		if(subject.isAuthenticated()) {
			request.getRequestDispatcher(loginSuccessJspUrl).forward(request, response);
			return;
		}
		
		String username = request.getParameter("username");
		String password = request.getParameter("password");
		UsernamePasswordToken token = new UsernamePasswordToken(username, password);
		
		String loginErrorMsg = null;
		
		try {
			subject.login(token);
		} catch(UnknownAccountException e) {
			loginErrorMsg = "查无此人！";
		} catch(IncorrectCredentialsException e) {
			loginErrorMsg = "密码错误！";
		} catch(AuthenticationException e) {
			loginErrorMsg = "登陆失败！";
		}
		
		request.setAttribute("loginErrorMsg", loginErrorMsg);
		
		if(subject.isAuthenticated()) {
			request.getRequestDispatcher(loginSuccessJspUrl).forward(request, response);
			return;
		} else {
			request.getRequestDispatcher(loginJspUrl).forward(request, response);
			return;
		}
	}

}
```

#### shiro.sql
```sql
drop database if exists shiro;
create database shiro;
use shiro;

create table users (
  id bigint auto_increment,
  username varchar(100),
  password varchar(100),
  password_salt varchar(100),
  constraint pk_users primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_users_username on users(username);

create table user_roles(
  id bigint auto_increment,
  username varchar(100),
  role_name varchar(100),
  constraint pk_user_roles primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_user_roles on user_roles(username, role_name);

create table roles_permissions(
  id bigint auto_increment,
  role_name varchar(100),
  permission varchar(100),
  constraint pk_roles_permissions primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_roles_permissions on roles_permissions(role_name, permission);

insert into users(username,password)values('luoyong','123');
insert into users(username,password)values('zhang', '456');

insert into user_roles (username, role_name) values ('luoyong', 'role1');
insert into user_roles (username, role_name) values ('zhang', 'role2');

insert into roles_permissions (role_name, permission) values ('role1', 'book:add');
insert into roles_permissions (role_name, permission) values ('role2', 'user:manager');
```