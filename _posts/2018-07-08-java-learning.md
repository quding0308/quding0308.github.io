---
layout: post
title:  "Java基础基础"
# date:   2018-07-09 10:000:00 +0800
categories: blog
---

 一些注意事项：
少用static，多用final

少些public，多写private

 

 

 import使用
 C++中的#include会把所包含的内容在编译时加入到文件中，java的import并不会。import和package机制相关，可以理解为：import是为了写代码时省略包名，少些一些代码，在编译时会把包名加上。

import会把public的类和接口导入
import不会导入子包，例如 import java.qd ，并不会导入  java.qd.test中的类
当前包中的类可以直接使用，不需要额外import
static import java.qd.ClassA.StaticField;   直接从某个class导入静态变量、长连、静态方法，
 

@interface自定义注解
使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译器完成其他工作。

@interface声明一个注解，每个方法实际是声明了一个参数。方法名对应参数名，返回值类型 对应参数类型。访问修饰符为 public和default。

支持的参数类型：

基础类型：int, float, double, boolean, byte, char, long, short
String, Class 
demo：﻿
 
        // 定义注解
        @Retention(RetentionPolicy.RUNTIME)
        @Target(ElementType.TYPE)
        @Documented
        @Inherited
        public @interface TaskProperty {
            String host() default "";
            String path();
            boolean shortChannelSupport() default true;
            boolean longChannelSupport() default false;
            String shortLinkUrl() default "";
            int cmdID() default -1;
        }

        // 使用
        @TaskProperty(
                host = "172.20.176.113",
                path = "/send_msg",
                cmdID = SEND_DATA_CMD,
                longChannelSupport = true,
                shortChannelSupport = true
        )
        class A {
            public A() {
                final TaskProperty property = this.getClass().getAnnotation(TaskProperty.class);
                if(property != null) {
                    //
                }
            }
        }