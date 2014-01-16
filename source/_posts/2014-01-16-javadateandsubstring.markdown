---
layout: post
title: "javaDateAndSubstring"
date: 2014-01-16 22:12
comments: true
categories: ["java","memo"]
---
substringの使い方と月末日の取得方法！

<!-- more -->
###substringの使い方
使い方はこんな感じ。
  
```
---------
substring
---------
public String substring(int beginIndex, int endIndex)
この文字列の部分文字列である新しい文字列を返す。  
部分文字列は、指定された beginIndex から始まり、インデックス endIndex - 1 にある文字まで。  
したがって、部分文字列の長さはendIndex-beginIndex になる。  
  
パラメータ:
  beginIndex - 開始インデックス (この値を含む。スタートは0から。)
  endIndex - 終了インデックス (この値を含まない) 
戻り値:
  指定された部分文字列 
例外: 
  IndexOutOfBoundsException - beginIndex が負の値である場合、endIndexが
  この String オブジェクトの長さより大きい場合、あるいはbeginIndex が endIndex より大きい場合
```
  
コードと実行結果。

```
class test{
  public static void main(String args[]){
	    String str1 = new String("123456789");
	    //3文字目から5文字目まで取得
	    String str1Substring = str1.substring(2, 5);

	    System.out.println("「" + str1 + "」のsubstring(2,5)は「" + str1Substring + "」です");

	    String str2 = new String("これはテストです");
	    //1文字目から4文字目まで取得
	    String str2Substring = str2.substring(0, 4);

	    System.out.println("「" + str2 + "」のsubstring(0,4)は「" + str2Substring + "」です");
	    
	    
  }
}
---実行結果---
「123456789」のsubstring(2,5)は「345」です
「これはテストです」のsubstring(0,4)は「これはテ」です
```
  
###年月日から月の月末日を取得
コードと実行結果  

```
import java.util.Calendar;

public class Test {

	public static void main(String[] args) {
		String str1 = "2014/01/01";
		String str2 = "2014/02/01";//notうるう年
		String str3 = "2016/02/11";//うるう年
		System.out.println(str1 + " の月末日は " +getLastDay(str1));
	    System.out.println(str2 + " の月末日は " +getLastDay(str2));
	    System.out.println(str3 + " の月末日は " +getLastDay(str3));
	}
	/**
	 * 指定した日付文字列（yyyy/MM/dd or yyyy-MM-dd）の月末日を返却する
	 * @param strDate 対象の日付文字列
	 * @return 月末日付
	 */
	public static int getLastDay(String strDate) {
	    if (strDate == null || strDate.length() != 10) {
	        throw new IllegalArgumentException("strDate：["+ strDate +"]" + "からは日付を取得できません!");
	    }
	    
	    int yyyy = Integer.parseInt(strDate.substring(0,4));//年取得
	    int MM = Integer.parseInt(strDate.substring(5,7));//月取得
	    int dd = Integer.parseInt(strDate.substring(8,10));//日取得
	    Calendar cal = Calendar.getInstance();
	    cal.set(yyyy,MM-1,dd);
	    int last = cal.getActualMaximum(Calendar.DATE);
	    return last;
	}
}
---実行結果---
2014/01/01 の月末日は 31
2014/02/01 の月末日は 28
2016/02/11 の月末日は 29
```
  
monthから1減算しているのは、下記の通り月の始まりが0からなので。  

```
------------
Calender.set
------------
public final void set(int year, int month, int date)
カレンダフィールド YEAR、MONTH、および DAY_OF_MONTH の値を設定します。他のカレンダフィール
ドの以前の値は保持されます。保持されないようにする場合は、最初に clear() を呼び出します。 

パラメータ:
  year - YEAR カレンダフィールドの設定に使用する値
  month - MONTH カレンダフィールドの設定に使用する値。Month 値は 0 から始まる (1月 は 0 になる)
  date - DAY_OF_MONTH カレンダフィールドの設定に使用する値
```