---
layout: post
title: "单例模式的两种实现方式"
category: 技术
tags: 设计模式
---

**经典实现**

	public class Singleton {
		private static Singleton instance = new Singleton();
		private Singleton() {
		}
		public static Singleton getInstance() {
			return instance;
		}
	}
	
以上这种方式也被称为“饿汉式”，还有另外一种“懒汉式”。“懒汉式”的出现主要是为了延迟加载，以提高效率和节约资源。不过懒汉式存在一定的线程安全问题，几经变革，最终形成了没有线程安全问题的版本，示例如下：

	public class Singleton {
		private Singleton(){};
		private static class SingletonHolder {
			static Singleton instance = new Singleton();
		}
		public static  Singleton getInstance(){
			return SingletonHolder.instance;
		}
	}
	
加载一个类时，其静态内部类不会同时被加载，只有当静态内部类的某个成员被调用时，他才会加载。