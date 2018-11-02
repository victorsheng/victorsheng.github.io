---
title: concurrent-program
abbrlink: 3350504067
date: 2018-10-23 18:28:59
tags:
categories:
---
译文:
http://ifeve.com/java-concurrency-thread-directory/
原文:
http://tutorials.jenkov.com/java-concurrency/index.html



# The "Double-Checked Locking is Broken" Declaration
```
public class Singleton {

    private static volatile Singleton instance = null;

    private Singleton(){}

   

    public static Singleton getInstance() {

       if(instance == null) {

           synchronized(Singleton.class) {

              if(instance == null) {

                  instance = new Singleton();

              }

           }

       }

       return instance;

    }

}
```
