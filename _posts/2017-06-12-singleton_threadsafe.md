---
layout: post
title: 单例模式的线程安全问题 
categories: [Android/ios]
tags: [编程细节]
description: 比较ios和Android中对单例线程安全的处理
---

tomorrow.cyz@gmail.com

# 单例模式
单例模式是非常常见的一种设计模式，因为他实现简单，并且可以全局共享，在开发中应用广泛。而且常见以懒汉模式方式实现。比如如下Java的实现
        
        public class SingletonObj{
            private static SingletonObj = null;
            private SingletonObj(){
                
            }
            public static SingletonObj getInstance(){
                if(instance == null){
                    instance = new SingletonObj();
                }
                return instance;
            }
        }
        //调用
        SingletonObj.getInstance();

# 线程安全
当多个线程都会调用到SingletonObj.getInstance()的时候，就需要考虑线程安全。如上的懒汉模式实现，getInstance这个接口是线程不安全的。
通常情况下，我倾向于利用java ClassLoader的线程安全机制，通过内部类来实现线程安全。
        
        public class SingletonObj{
            private static class SingletonHolder{
                private static SingletonObj instance = new SingletonObj();
            }
            private SingletonObj(){
                
            }
            public static SingletonObj getInstance(){
                return SingletonHolder.instance;
            }
        }

# ios单例模式线程安全的处理
Java中单例的线程安全处理，其实工作中会发现很多初级/中级工程师处理不好。ios则通过一般GCD(Grand Central Dispactch)的机制来实现的单例。
显然给工程师留下的犯错空间大大减少。最近学习ios，发现ios很多地方都设计得让开发者减少犯错机会，比如alloc的时候强制需要init，这种设计
思路在编写给外部模块用的代码的时候值得推广。
        
        +(SchoolManager *)sharedInstance  
        {  
            static SchoolManager *sharedManager;  
        
            static dispatch_once_t onceToken;  
            dispatch_once(&onceToken, ^{  
                    sharedManager = [[SchoolManager alloc] init];  
                    });  
        
            return sharedManager;  
        }   
