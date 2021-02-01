# RBAC简介：
Role-Based Access Control，基于角色的访问控制。一种思想，根据RBAC思想进行数据库设计，根据数据库设计更好的完成权限控制。

1. 作用：实现访问控制
2. 核心：角色

### 权限控制常用分类：
1. 菜单控制
2. url控制（控制访问不同的控制器）
3. 资源可见性 （页面中某些元素对不同用户的可见性是不同的）

### 发展历史：
1. RBAC1.0：通过 **角色表** 来管理权限。使用RBAC思想实现步骤：

   * 如果一个用户只能拥有一个角色，可以在用户表中直接添加外键列，直接应用角色表
   * 如果每个用户可能有多个角色，可以用户-角色表中添加用户和角色的关系
     * 建立一个用户角色表映射用户和角色之间的关系
     * 建立角色菜单表映射角色和菜单的关系
     * **用户表** -- **用户菜单表** -- **角色表** -- **角色菜单表** -- **菜单表**
2. RBAC2.0：项目比较大时使用。

## 根据RBAC进行数据库设计（1.0）

```sql
-- 角色表
create table rbac_roles(
id int(10) PRIMARY key auto_increment,
name varchar(20)
)
insert into rbac_roles VALUES(DEFAULT,'管理员');
insert into rbac_roles VALUES(DEFAULT,'销售经理');

-- 用户表
create table rbac_users(
id int(10) primary key auto_increment,
username varchar(20),
password varchar(20),
rid int(10)
)
insert into rbac_users values (default,'a','a',1);
insert into rbac_users values (default,'b','b',2)

-- 下面是希望添加的功能
-- 1. 新建功能
create table rbac_menu(
id int(10) primary key auto_increment,
name varchar(10),
pid int(10)
);

insert into menu VALUES(DEFAULT,'系统设置',0)
insert into menu VALUES(DEFAULT,'销售管理',0);
insert into menu VALUES(DEFAULT,'部门管理',1);
insert into menu VALUES(DEFAULT,'角色管理',1);
insert into menu VALUES(DEFAULT,'权限管理',1);
insert into menu VALUES(DEFAULT,'订单管理',2);
insert into menu VALUES(DEFAULT,'客户管理',2);

-- 2. 和角色表产生关系
create table rbac_role_menu(
id int(10) PRIMARY key auto_increment,
rid int(10),
mid int(10)
);

insert into rbac_role_menu VALUES(default,1,1);
insert into rbac_role_menu VALUES(default,1,2);
insert into rbac_role_menu VALUES(default,1,3);
insert into rbac_role_menu VALUES(default,1,4);
insert into rbac_role_menu VALUES(default,1,5);
insert into rbac_role_menu VALUES(default,1,6);
insert into rbac_role_menu VALUES(default,1,7);
insert into rbac_role_menu VALUES(default,2,2);
insert into rbac_role_menu VALUES(default,2,6);
insert into rbac_role_menu VALUES(default,2,7);
```

## 使用RBAC进行页面元素可见性控制
1. 添加页面元素表
	```sql
	create table element(
	id int(10) primary key auto_increment,
	elemo varchar(20)
	)
	-- 在表中添加一个授权按钮
	insert into element values(default,'grant');
	```
2. 为该表和角色表添加关联
	```sql
	create table role_element(
	id int(10) primary key auto_increment,
	rid int(10),
	eid int(10)
	);
	insert into role_element values(default,1,1);
	```
3. 在项目中创建页面元素实体类，sql接口，service类，并在用户类中添加元素类的列表
4. 调用service的方法，获取该用户有哪些element
5. jsp中通过c:foreach判断该用户是否有相应的权限，并确定是否显示

## 使用RBAC实现url控制
1. 进行数据库设计
	```sql
	-- 1. 把需要进行权限控制的url添加到表中
	create table url(
	id int(10) primary key auto_increment,
	name varchar(100),
	);
	-- 添加要进行权限控制的url到表中，可以是jsp，或者servlet
	insert into url values(default,'/');
	insert into url values(default,'/');

	-- 2. 为该表和角色表添加关联
	create table role_url(
	id int(10) primary key auto_increment,
	rid int(10),
	uid int(10)
	);
	insert into role_url values(default,1,1);
	insert into role_url values(default,2,2);
	```
3. 新建一个url实体类并在用户类中添加该实体类的一个List
	```java
	class Url{
		private int id;
		private String name;
		···
	}
	Class Users{
		···
		List<Url> url;
		···
	}
	```
4. 新建一个Url类的mapper
	```java
	Interface UrlMapper{
		@Select("select * from url where id in(select uid from role_url where rid=#{0}")
		List<Url> selByRid(int rid);
		@Select("select * from url")
		List<Url> selAll();
	}
	```
5. 新建一个UrlService接口并实现该接口
6. 在登陆控制器中使用该接口的方法查询用户的url，并放入用户类中
7. 使用控制器进行拦截：
	```java
	String uri = req.getRequestURI();
	if(uri.endsWith(".js")||uri.endsWith(".css")||uri.endsWith(".png")||uri.endsWith(".html")){
		//放行js,css,图片
		chain.doFilter(req,resp);
	}else if(uri.equals("/login")||uri.equals("/login.jsp")){
		//放行登陆请求
		chain.doFilter(req,resp);
	}else{
		//对需要进行权限控制的url进行判断
		HttpSession session = request.getSession();
		Object obj = session.getAttribute("user");
		if(obj!=null){
			//登陆不为空的情况
			List<Url> allUrls = (List<Url>) session.getAttribute("allUrl");
			boolean isExists = false;
			for(Url url : allUrls){
				//判断是否需要权限控制
				if(url.getName().equals(uri))
					isExists = true;
			}
			//如果需要进行权限控制
			if(isExists){
				User user = (User)obj;
				boolean isRight = false;
				for(Url url:users.getUrls()){
					//判断用户是否对url有访问权限
					if(url.getName().equals(uri))
						isRight = true;
				}
				if(isRight){
					chain.doFilter(req,resp);
				}else{
					session.removeAttribute("user");
					session.removeAttribute("allUrl");
					req.sendRedirect("/rbrc/login.jsp")
				}
			}else{
				chain.doFilter(req,resp);
			}
		}else{
			//没有登陆
			req.sendRedirect("/rbrc/login.jsp")
		}
		
	}
	```