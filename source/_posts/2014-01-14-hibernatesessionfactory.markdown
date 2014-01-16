---
layout: post
title: "hibernateSessionFactory"
date: 2014-01-14 02:01
comments: true
categories: hibernate
---
###SessionFactoryをシングルトン化する
データ永続化のPOJOは作成しているものとする。  
ここでそれぞれのDAOにはいくつか共通するコードが出てくる。  
特にセッション取得やクローズを毎回記述するのは面倒ですので、以下のようにDAOの基底クラスを作成し、
共通メソッドを定義します。  
<!-- more -->
  
セッションファクトリ管理クラス：SessionManager.java  
基底クラス					：BaseDAO.java  
個別DAO					：それぞれ  
  
セッションファクトリ管理クラス  

```
import org.hibernate.HibernateException;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

/**
 * HibernateのSessionFactoryを保持するシングルトン
 */
public class SessionManager {
    private static SessionManager instance;

    private SessionFactory sessions;

    private SessionManager() throws HibernateException {
    	Configuration configuration = new Configuration();
    	sessions = configuration.configure().buildSessionFactory();
    }

    public static synchronized SessionManager getInstance() {
        if (instance == null) throw new NotInitializedException();
        return instance;
    }

    /**
     * SessionFactoryを初期化する。
     * getInstance()を呼び出す前に必ず一度呼び出す必要がある。
     * @throws HibernateException
     */
    public static synchronized void initialize() throws HibernateException {
        if (instance == null)
            instance = new SessionManager();
    }
    
    public Session getSession() throws HibernateException {
        return sessions.openSession();
    }

    public static class NotInitializedException extends RuntimeException {}
}
```
  
基底クラス  

```
import org.hibernate.HibernateException;
import org.hibernate.Session;
import org.hibernate.Transaction;

/**
 * 各DAOにSession取得のための簡易メソッドを提供する基底クラス。
 */
public class BaseDAO {
    protected SessionManager manager;
    
    public BaseDAO() {
    	//セッションマネージャー初期処理
		SessionManager.initialize();
    	manager = SessionManager.getInstance();
    }
    
    /** 
     * HibernateのSessionを取得する
     * @return
     * @throws HibernateException
     */
    public Session getSession() throws HibernateException {
        return manager.getSession();
    }
    
    /** 
     * トランザクションのロールバックを行う
     * @param session
     */
    public void rollback(Transaction tr) {
        try {
            tr.rollback();
        } catch (HibernateException ignore) {}
    }
    
    /**
     * HibernateのSessionを閉じる
     */
    public void closeSession(Session session) {
        if (session != null) {
            try {
                session.close();
            } catch (HibernateException ignore) {}
        }
    }
}
```
  
個別DAO  

```
package test.dao;

import java.util.List;

import org.hibernate.HibernateException;
import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.Transaction;

/**
 * Testテーブルへのアクセスを行うDAO
 */
public class sampleDao extends BaseDAO {
    
	    /**
	     * データベースへオブジェクトをinsertする
	     */
	    public void save(Test obj) throws HibernateException {
	        Session sess = getSession();
	        Transaction tx = null;
	        try {
	            tx = sess.beginTransaction();
	            sess.save(obj);
	            tx.commit();
	        } catch (HibernateException ex) {
	            rollback(tx);
	            throw ex;
	        } finally {
	            closeSession(sess);
	        }
	    }

	    /**
	     * データベース内のオブジェクトを更新する
	     */
	    public void update(Test obj) throws HibernateException {
	        Session sess = getSession();
	        Transaction tx = null;
	        try {
	            tx = sess.beginTransaction();
	            sess.update(obj);
	            tx.commit();
	        } catch (HibernateException ex) {
	            rollback(tx);
	            throw ex;
	        } finally {
	            closeSession(sess);
	        }
	    }

	    /**
	     * 引数で指定した主キー値にマッチするTestオブジェクトを取得する
	     * @param id 複合主キークラス
	     */
	    public Test load(test.dao.TestId id) throws HibernateException {
	    	Test user = null;
	        Session sess = getSession();
	        try {
	            user = (Test) sess.get("test.dao.Test", id);
	        } finally {
	            closeSession(sess);
	        }
	        return user;
	    }

	    /**
	     * 引数にマッチする名前を持つTestオブジェクトのリストを取得する
	     */
	    public List<Test> findByName(String name) throws HibernateException {
	        List<Test> test = null;
	        Session sess = getSession();
	        
	        //HQL文  
	        String hql = "FROM test.dao.Test as test WHERE test.name = :name";  
	        
	        try {
	        	Query query = sess.createQuery(hql);  
	      	  
		        //条件          
		        query.setString("name", new String(name));  
		          
		        //クエリ発行
		        test = (List<Test>) query.list();
	        } finally {
	            closeSession(sess);
	        }
	        return test;
	    }
	
}
```
  
あとは呼び出すクラス  

```
import java.util.List;

public class TestSample {

	public static void main(String[] args) {
		TestId id = new TestId(1,"admin");
		System.out.println("Factoryをシングルトンにした場合");
		sampleDao s = new sampleDao();
		System.out.println("主キーによる検索結果");
		System.out.println(s.load(id));
		
		System.out.println("nameによる検索");
	    	
		for (Test t : s.findByName("てすと１")) {
			System.out.print(t.getId().getId() + " ");
			System.out.print(t.getId().getUserid() + " ");
			System.out.println(t.getName());
		}
		
		System.out.println("主キーの一部による検索");
		for (Test t : s.findByUserId("admin")) {
			System.out.print(t.getId().getId() + " ");
			System.out.print(t.getId().getUserid() + " ");
			System.out.println(t.getName());
		}
	
	}

}
```