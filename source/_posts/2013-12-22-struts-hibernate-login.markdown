---
layout: post
title: "Struts2とHibernate4を使ってTodoアプリをつくろう その１"
date: 2013-12-22 23:35
comments: true
categories: ["勉強","Struts","hibernate"]
---
###事の始まり
12月半ばにとあるプロジェクトにアサインされた。どうやら新規開発らしい。  
開発案件から遠ざかっていたし、２年ほどいた職場で病んでいた精神も薬とかカウンセリングとかで
やっとこさマシな状況まで回復してきたので、開発に復活するにはちょうど良さそうなタイミングだった。  
2年半ほど、(なんちゃって)運用系な作業をやっていたので、開発は2年半ぶりｗ  
いやー楽しみですわー
  
MVCはStruts2で、O/RMapperはHibernate使うから勉強しといて！！
  
<!-- more -->
####どっちも触ったことねーーー(´・ω・｀)
Struts1.x系はちょろっと改修するときに見たことある程度だったので、マジ初心者。  
Hibernate？名前しか知りませんがなにか？←  
という残念っぷり。。。  
というわけで、勉強も兼ねて、Struts2.x系とHibernate4.x系使ってTODOアプリでも作ってみるかー。  
  
眠いのでまとめるのはまた今度ちゃんとやる。  

#限りなく自分用のメモです。
  
###環境

1. MacBookAir
2. Mysql5.6
3. Struts2.3.16
4. Hibernate4.3.0
5. Tomcat7.0.47
6. Eclipse kepler
	+ Tomcatプラグイン3.3
	+ Hibernate Tools
		* Hibernate Toolsは、Eclipseで`http://download.jboss.org/jbosstools/updates/stable/kepler/`を指定して、「Hibernate Tools」でフィルター後、複数出てくるが１番上のを入れてみた


###Strutsの設定周り
web.xml
filter-classへの指定がどっかのバージョンから変わったらしい。。。
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app id="WebApp_9" version="2.4" xmlns="http://java.sun.com/xml/ns/j2ee" 
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

    <display-name>Struts Todo</display-name>
    
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

</web-app>
```
  
struts.xml
ここら辺は特に変化はないはず。
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
	"-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
	"http://struts.apache.org/dtds/struts-2.3.dtd">

<struts>
    <package name="StrutsTodoProject" extends="struts-default">

        <action name="Login" class="todo.Login">
            <result name="success">ぺーじ</result>
        </action>

        <action name="LoginConfirm" method="confirm" class="todo.Login">
            <result name="success">ぺーじ</result>
            <result name="error">ぺーじ</result>
            <result name="input">ぺーじ</result>
        </action>
    </package>
</struts>
```
  
あとは、formのとこでなぜか詰まってしまった。。。  
formのactionには、struts.xmlのactionタグのnameを書く。  
参考にしてたサイトがsubmitのとこに入れてたので、動かんくて困った。。。  
```
<body>
	<s:form action="LoginConfirm" validate="true">
		<s:textfield label="〜" name="xxxx" />
		<s:password label="〜" name="yyyy" />
		<s:submit value="ボタン" />
	</s:form>
</body>
```
  
###Hibernate
ここを参考に[HibernateToolsで簡単Hibernate活用](http://www.omotenashi-mind.com/index.php/Hibernate_Tools%E3%81%A7%E7%B0%A1%E5%8D%98Hibernate%E6%B4%BB%E7%94%A8)ツールのインストール。  
Hibernate Reverse Engineering File以降は、テーブルを作ってからやる（当然）。
  
hibernate.cfg.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
                                         "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
	<session-factory name="">
		<property name="hibernate.connection.driver_class">org.gjt.mm.mysql.Driver</property>
		<property name="hibernate.connection.url">jdbc:mysql://localhost:3306/でーびー</property>
		<property name="hibernate.connection.username">ゆーざー</property>
		<property name="hibernate.connection.password">ぱすわーど</property>
		<property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
		<property name="hibernate.connection.pool_size">3</property>
		<property name="hibernate.current_session_context_class">thread</property>
		<property name="hibernate.show_sql">true</property>
		<!-- Mapping files -->
		<mapping  resource="まっぴんぐふぁいるのありか" />
	</session-factory>
</hibernate-configuration>
```
  
HibernateToolsで作成したDAO(一部修正)
```
/**
 * Home object for domain model class TodoUser.
 * @see todo.hibernate.TodoUser
 * @author Hibernate Tools
 */
public class TodoUserHome {

	private static final Log log = LogFactory.getLog(TodoUserHome.class);
	private SessionFactory sessionFactory;
	
	public TodoUserHome() {
		HibernateUtil h = new HibernateUtil();
		sessionFactory = h.getSessionFactory();
	}

    /** 
     * セッションを取得する 
     * @return Session 
     */
    // TODO 共通化すること 
    private Session getSession() {  
    	Session session = sessionFactory.getCurrentSession();
        session.beginTransaction();
        return session;  
    } 
    
	/**
	 * 
	 * @param transientInstance
	 */
	public void persist(TodoUser transientInstance) {
		log.debug("persisting TodoUser instance");
		try {
			Session s = getSession();
			s.persist(transientInstance);
			log.debug("persist successful");
		} catch (RuntimeException re) {
			log.error("persist failed", re);
			throw re;
		}
	}

	/**
	 * 登録or更新
	 * @param instance
	 */
	public void attachDirty(TodoUser instance) {
		log.debug("attaching dirty TodoUser instance");
		try {
			Session s = getSession();
			s.saveOrUpdate(instance);
			s.getTransaction().commit();
			log.debug("attach successful");
		} catch (RuntimeException re) {
			log.error("attach failed", re);
			throw re;
		}
	}

	/**
	 * 
	 * @param instance
	 */
	public void attachClean(TodoUser instance) {
		log.debug("attaching clean TodoUser instance");
		try {
			Session s = getSession();
			s.lock(instance, LockMode.NONE);
			log.debug("attach successful");
		} catch (RuntimeException re) {
			log.error("attach failed", re);
			throw re;
		}
	}

	/**
	 * 削除
	 * @param persistentInstance
	 */
	public void delete(TodoUser persistentInstance) {
		log.debug("deleting TodoUser instance");
		try {
			Session s = getSession();
			s.delete(persistentInstance);
			s.getTransaction().commit();
			log.debug("delete successful");
		} catch (RuntimeException re) {
			log.error("delete failed", re);
			throw re;
		}
	}
	
	/**
	 * 登録処理
	 * @param instance
	 */
	public void save(TodoUser instance) {
		log.debug("Save TodoUser instance");
		try {
			Session s = getSession();
			s.save(instance);
			s.getTransaction().commit();
	        log.debug("save successful");
		} catch (RuntimeException re) {
			log.error("save failed", re);
			throw re;
		}
	}
	
	public TodoUser merge(TodoUser detachedInstance) {
		log.debug("merging TodoUser instance");
		try {
			TodoUser result = (TodoUser) sessionFactory.getCurrentSession()
					.merge(detachedInstance);
			log.debug("merge successful");
			return result;
		} catch (RuntimeException re) {
			log.error("merge failed", re);
			throw re;
		}
	}
	
	/**
	 * IDによる検索
	 * @param id
	 * @return
	 */
	public TodoUser findById(java.lang.Integer id) {
		log.debug("getting TodoUser instance with id: " + id);
		try {
	    	Session session = sessionFactory.getCurrentSession();
	        session.beginTransaction();
	        TodoUser instance = (TodoUser) session.get("todo.hibernate.TodoUser", id);
			if (instance == null) {
				log.debug("get successful, no instance found");
			} else {
				log.debug("get successful, instance found");
			}
			return instance;
		} catch (RuntimeException re) {
			log.error("get failed", re);
			throw re;
		}
	}
	
	@SuppressWarnings("unchecked")
	public List<TodoUser> findByExample(TodoUser instance) {
		log.debug("finding TodoUser instance by example");
		try {
			List<TodoUser> results = (List<TodoUser>) sessionFactory
					.getCurrentSession()
					.createCriteria("todo.hibernate.TodoUser")
					.add(create(instance)).list();
			log.debug("find by example successful, result size: "
					+ results.size());
			return results;
		} catch (RuntimeException re) {
			log.error("find by example failed", re);
			throw re;
		}
	}
	
    /** 
     * TodoUserデータ検索（全件取得）
     * @return TODOUSERの全件データ。 
     */  
    @SuppressWarnings("unchecked")
	public List <TodoUser> getAllList() {  
        return getSession().createCriteria(TodoUser.class).list();  
    }

    /** 
     * HQL文より指定条件で合致するユーザのリストを取得する
     * @param name ユーザ名
     * @return 合致するユーザのリスト
     */
    @SuppressWarnings("unchecked")
	public List <TodoUser> getListByNameHql(String name) {  
        //HQL文  
        String hql = "FROM todo.hibernate.TodoUser as u WHERE u.name = :name";  
        Query query = getSession().createQuery(hql);  
  
        //条件          
        query.setString("name", name);  
          
        //クエリ発行  
        return query.list();  
    }  
   
    /** 
     * HQL文より指定条件で合致するユーザ名とパスワードのリストを取得する
     * @param name ユーザ名
     * @param password パスワード
     * @return 合致するユーザのリスト
     */   
    @SuppressWarnings("unchecked")
    public List <TodoUser> getListByNamePassowordHql(String name, String password) {  
        //HQL文  
        String hql = "FROM todo.hibernate.TodoUser as u WHERE u.name = :name AND u.password = :password";  
        Query query = getSession().createQuery(hql);  
  
        //条件          
        query.setString("name", name);  
        query.setString("password", password);  

        //クエリ発行  
        return query.list();  
    }  
    
    /** 
     * SQL文より指定条件で合致するユーザのリストを取得する
     * @param name ユーザ名 
     * @return 合致するユーザのリスト 
     */  
    @SuppressWarnings("unchecked")
	public List <TodoUser> getListByNameSql(String name) {  
        //HQL文  
        String sql = "FROM TODOUSER u WHERE u.NAME = :name";  
        Query query = getSession().createSQLQuery(sql);  
  
        //条件          
        query.setString("name", name);  
          
        //クエリ発行  
        return query.list();  
    }
}
```
  
HibernateUtil.java
```
package todo.hibernate;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {

    private final SessionFactory sessionFactory = buildSessionFactory();

    @SuppressWarnings("deprecation")
	private SessionFactory buildSessionFactory() {
        try {
            // Create the SessionFactory from hibernate.cfg.xml
            return new Configuration().configure().buildSessionFactory();
        }
        catch (Throwable ex) {
            // Make sure you log the exception, as it might be swallowed
            System.err.println("Initial SessionFactory creation failed." + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }

    public SessionFactory getSessionFactory() {
        return sessionFactory;
    }

}
```
  
###どうしてこうなった
なぜかしょうもないとこ含め、色んなとこで詰まってしまった(´・ω・｀)  
まぁ、これも経験ということで。やっぱり手を動かさないと覚えないもんだよねー。  
