---
layout:         post
title:          java native
subtitle:       java native 笔记
date:           2019-03-13 17:54:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - android
---

* content
{:toc}

## 开发步骤
- public class A {public static native String hello(String name);}
- javac src/com/study/jnilearn/A.java -d ./bin
- javah -jni -classpath ./bin [-d ./jni | -o ./jni/A.h] com.nomdli.A
- 实现.h头文件中的函数
- 编译成动态库(*.dll *.so .jnilib)
- static{System.loadLibrary("A");} 或 static{System.load("/x/x/A.so");}

## C方法参数及类型
- 第一个参数JNIEnv* java虚拟机上下文,是JavaVM下每个线程一个。JavaVM代表虚拟机
- 第二个参数,如果是成员方法为jobject即类实例对象,如果是静态方法则为jclass即类本身
- 后面为定义参数
- 基本数据类型[jbyte|jchar|jshort|jint|jlong|jfloat|jdouble|jboolean]
- 类类型[jclass|jobject|jstring|jarray|jthrowable]
- jarray子类[jbyteArray|jcharArray|jshortArray|jintArray|jlongArray|jfloatArray|jdoubleArray|jbooleanArray|jobjectArray]
- jbyte = signed char
- jchar = unsigned short
- jshort = signed short
- jint = signed int32
- jlong = signed int64
- jfloat = float32
- jdouble = float64
- jboolean = unsigned char
- 编译器为c++则使用类,否则使用c的联合将各个结构统一为jvalue类型
- 类类型对象为堆对象,无法直接访问,需要通过env上下文调用方法拷贝到c堆中,因此需要内存释放

## jstring
- 获取虚拟机内string的指针,会触发暂停GC, 拷贝不会暂停GC,因此是指针时,在释放之前不要再次调用env方法、也不要阻塞
- GetStringRegion|GetStringUTFRegion                    直接拷贝到c自己提供的内存,无需释放
- GetStringChars|ReleaseStringChars                     Unicode格式(U16)
- GetStringUTFChars|ReleaseStringUTFChars               UTF-8
- GetStringCritical|ReleaseStringCritical               希望尽量返回源的指针,而不是拷贝
- GetStringLength                                       获取jstring长度
- GetStringUTFLength                                    获取UTF-8长度 = strlen
```
    jstring jinstr;
    jboolean iscopy;//JNI_TRUE 源字符拷贝, JNI_FALSE 源字符指针
    const char *str = (*env)->GetStringUTFChars(env, jinstr, &iscopy);
    if (str == NULL) {...}
    (*env)->ReleaseStringUTFChars(env, jinstr, str);

    char buf[128];
    (*env)->GetStringUTFRegion(env, jinstr, 0, 127, buf);
    buf[127] = '\0';

    jstring ret = (*env)->NewStringUTF(env, buf);
``` 

## jarray 基本数据
- GetArrayLength                                        获取元素个数
- Get[Int]ArrayRegion                                   将指定范围存入c-buf
- Set[Int]ArrayRegion                                   修改指定范围的数据
- Get[Int]ArrayElements|Release[Int]ArrayElements       直接返回数据,需要释放,可能是直接指针
- Get/ReleasePrimitiveArrayCritical                     尽量返回直接指针
```
   jint len = (*env)->GetArrayLength(env, jintarray);
   (*env)->GetIntArrayRegion(env, jintarray, 0, len, c-buf);

   c-p = (*env)->GetIntArrayElements(env, jintarray, &iscopy);
   (*env)->ReleaseIntArrayElements(env, jintarray, c-p, 0); 
```

## jarray 对象
- GetObjectArrayElement                                 获取对象
- SetObjectArrayElement                                 修改对象
```
    jclass intclass = (*env)->FindClass(env,"[I");              //int数组类型
    jobjectArray arry = (*env)->NewObjectArray(env, size, intclass, NULL);//新建存int数组的数组
    jintArray intArr = (*env)->NewIntArray(env, size);          //新建int数组
    (*env)->SetIntArrayRegion(env, intArr, 0, size, buff);      //赋值
    (*env)->SetObjectArrayElement(env, arry, i, intArr);        //将int数组插入数组      
    (*env)->DeleteLocalRef(env, intArr);                        //解引用计数
    (*env)->DeleteLocalRef(env, intclass);                      //解引用类对象
```

## 方法签名
- (param1;param2;..)return
- 类型为引用需要使用L开头,并且指定全路径如 Ljava/lang/String
java type| field
|:-|:-:|
boolean|Z
byte|B
char|C
short|S
int|I
long|J
float|F
double|D
Class|java/lang/Class
String|java/lang/String

## 调用java 类函数
- CallStatic[Void|Int|Float|Short|Object|...]Method      [表示返回类型] 可变参数,根据定义
- CallStatic[...]MethodV                                 第四参数为va_list 
- CallStatic[...]MethodA                                 第四参数为const jvalue*
```
    jclass c = (*env)->FindClass(env,"com/nomadli/xxx");                        //找到类对象
    jmethodID static_func = (*env)->GetStaticMethodID(env, c, "classfunc", "(Ljava/lang/String;I)V"); //获取类方法
    jstring str = (*env)->NewStringUTF(env, "call class func");                 //构造参数
    (*env)->CallStaticVoidMethod(env, jclass, static_func, str, 100);           //调用类方法
    (*env)->DeleteLocalRef(env, c);                                             //解引用类对象
    (*env)->DeleteLocalRef(env, str);                                           //解引用参数
```

## 调用java 成员函数
- Call[Void|Int|Float|Short|Object|...]Method           [表示返回类型] 可变参数,根据定义
- Call[...]MethodV                                      第四参数为va_list 
- Call[...]MethodA                                      第四参数为const jvalue*
```
    jclass c = (*env)->FindClass(env,"com/nomadli/xxx");                        //找到类对象
    jmethodID init_func = (*env)->GetMethodID(env, c, "<init>", "()V");         //获取构造函数
    jmethodID cfunc = (*env)->GetMethodID(env, c, "c_func", "(Ljava/lang/String;I)V");//获取成员函数
    jobject cinstance = (*env)->NewObject(env, c, init_func);                   //新建实例对象
    jstring str = (*env)->NewStringUTF(env, "call class func");                 //构造参数
    (*env)->CallVoidMethod(env, cinstance, cfunc, str, 100);                    //调用实例方法
    (*env)->DeleteLocalRef(env, c);                                             //解引用类对象
    (*env)->DeleteLocalRef(env, str);                                           //解引用参数
    (*env)->DeleteLocalRef(env, cinstance);                                     //解引用实例对象
``` 

## 调用java 类变量
```
    jclass c = (*env)->FindClass(env, "com/nomadli/xxx");                        //找到类对象
    jfieldID fid = (*env)->GetStaticFieldID(env, c, "str", "Ljava/lang/String;");//获取类象的类变量str
    jstring str = (jstring)(*env)->GetStatic[Object]Field(env,clazz,fid);       //获取类象的类变量的值
    (*env)->DeleteLocalRef(env, c);                                             //解引用类对象
    (*env)->DeleteLocalRef(env, str);                                           //解引用值
    (*env)->DeleteLocalRef(env, fid);                                           //解引用变量
```

## 调用java 成员变量
```
    jclass c = (*env)->GetObjectClass(env, jobject);                            //获取实例对象的类
    jfieldID fid = (*env)->GetFieldID(env, c, "str", "Ljava/lang/String;");     //获取实例对象的成员变量str
    jstring str = (jstring)(*env)->Get[Object]Field(env, c, fid);               //获取实例对象成员变量的值
    (*env)->SetObjectField(env, c, fid, jstring);                               //修改实例对象成员变量的值
    (*env)->DeleteLocalRef(env, c);                                             //解引用类对象
    (*env)->DeleteLocalRef(env, str);                                           //解引用值
    (*env)->DeleteLocalRef(env, fid);                                           //解引用变量
```

## 高级特性
- AllocObject                       //可以创建一个未初始化的对象
- CallNonvirtual[Void|...]Method    //调用指定类的指定函数[...]返回类型,可以掉用父类函数
- java 调用System.loadLibrary()时会调用C的JNI_OnLoad函数
```
    jint JNI_OnLoad(JavaVM *vm,void *reserved) {
        JNIEnv *env = NULL;
        if (vm->GetEnv((void**)(&env), JNI_VERSION_1_6) != JNI_OK) {
            return JNI_ERR;         //不支持JIN 1.6
        }

        JNINativeMethod methods[] = {
            {"java_hello","(Ljava/lang/String;)V", (void*)c_hello}
        };
        int count = 1;
        jclass c = env->FindClass(com/nomadli/xxx);
        result = env->RegisterNatives(c, methods, count);
        if(result != count){
            return JNI_FALSE;
        }

        if (vm->AttachCurrentThread(&env, 0) != 0) {
            //获取当前线程的env失败
        }
        vm->DetachCurrentThread();//是否env
        DestroyJavaVM(vm);// 卸载虚拟机, 这真的会把java关掉


        return JNI_VERSION_1_6;
    }

```