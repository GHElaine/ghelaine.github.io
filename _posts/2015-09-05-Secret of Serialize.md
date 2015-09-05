---
layout : default
title: 序列化的秘密-读书笔记（1）
---  
# 序列化的秘密--读书笔记（1）
> Edit by Yuanqi
> 2015-09-05  

  
## 为什么要有序列化   
 
### 序列化解决的问题   
*    内存被回收后，内存中的对象没法用了  
*    网络上传输的对象，而网络两端不是不共享内存的  
*    远程接口调用，两端各自再虚拟机中运行，内存不共享，参数怎么传，结果怎么返   
*    。。。。。。  
   
#### 序列化就是把这些对象转化成一段字节序列，放在流中，可存储于文件或在网络传输，需要用到时读取这个字节序列，重建对象    

## 序列化可能引起的问题  
*    序列化的实例创建不受限制   
*    序列化的实例可能无法符合类之前的一些要求 ，因为不调用类的构造器   
  
### 一个类，哪些东西需要序列化    
*    静态成员是类属性，不是对象属性，***不需要***实例化    
*    普通成员变量***需要***实例化
*    成员方法是类的无状态指令，***不需要***序列化   
*    成员是引用变量，需要序列化，保证变量的信息完整  
*    继承自序列化过的父类：父类会被序列化，子类不显式表明实现序列化接口也依然可以被序列化，反序列化时不会调用父类的构造方法   
*    继承自未实现序列化的父类： 
    1. 父类有无参构造方法，子类可序列化，且反序列化时调用父类无参构造方法，但父类不可序列化；   
    2. 父类没有无参构造方法，子类反序列化时会抛异常，因为调不到父类的构造方法，无法完整还原对象信息      

### 需要被保护的信息    
使用transient关键词修饰。  
由transient修饰的变量，不会被序列化。  

### 序列化算法    
1.    文件头，表明协议和版本号，类似名片的作用。  
2.    对象信息。类名，有哪些属性。  
3.    当前对象属性信息，什么类型，什么名字。  
3.    当前类的父类信息，类名、属性。   
4.    父类的属性值。
5.    当前对象的属性值。  
6.    当前对象的引用类信息，属性信息，包括其父类。循环2-5的过程。
7.    当前对象的引用类属性值。   
栈式存储。反序列化时，先实例化父类，再实例化子类，因此父类信息在栈顶。   

## 定制个性化序列化算法  
  
#### 默认序列化算法不适用的场景：   
*    安全性高 构造放阿飞的约束严格的场景  
*    类很大，但只想保存一部分信息   不想用transient一个一个标记  
*    希望序列化后的文件易读易懂   
*    逻辑结构（以对象维根的结构）和物理结构（对象序列化后的结构）相差太远     

#### 定制序列化算法的方式：    
*    实现了Serializable的类，覆写writeObject()和readObject()  

  
```     

     import java.io.*;   
     /**   
      * Created by elaine on 9/5/15.   
      */    
    public class StringList implements Serializable {   
        private static final Long serialVersionUID = 1L;   
                
    transient private int size = 0;
    transient private Entry head = null;

    private static class Entry implements Serializable {
        private static final long serialVersionUID = 1L;
        String data;
        Entry next;
        Entry pre;
    }

    public StringList(String headString) {
        initHead(headString);
    }

    private void initHead(String headString) {
        this.head = new Entry();
        this.head.data = headString;
        this.head.next = this.head;
        this.head.pre = this.head;
        this.size = 1;
    }

    public StringList add(String str) {
        Entry newEntry = new Entry();
        newEntry.data = str;
        newEntry.pre = this.head.pre;
        this.head.pre.next = newEntry;
        this.head.pre = newEntry;
        newEntry.next = this.head;

        this.size ++;
        return this;
    }

    public String toString() {
        Entry e = head;
        StringBuilder sb = new StringBuilder(e.data);
        while (hasNext(e)) {
            e = e.next;
            sb.append("->" + e.data);
        }
        return sb.toString();
    }

    private boolean hasNext(Entry e) {
        return e.next != this.head;
    }

    private void writeObject(ObjectOutputStream s) throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);

        // 写入头数据
        Entry e = head;
        s.writeUTF(e.data);

        // 按顺序写入其他字符串
        while(hasNext(e)) {
            e = e.next;
            s.writeUTF(e.data);
        }
    }

    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int size = s.readInt();

        String headStr = s.readUTF();
        initHead(headStr);

        for (int i = 1; i < size; i++) {
            String str = s.readUTF();
            add(str);
        }
    }
    public static void main(String[] arg) throws IOException, ClassNotFoundException{
        StringList sl = new StringList("yuanqi");
        sl.add("is").add("Elaine");
        System.out.println(sl);
        SerialUtils.writeObject(sl, "sl.byte");
        SerialUtils.readObject("sl.byte");
    }
}   
```


*    实现Externalizable接口     
        Externalizable接口继承自Serializable，区别如下：  
    *    Externalizable里的writeExternal 、readExternal两个方法必须实现    
    *    Serializable在反序列化时，生成实例依靠语言外的隐藏构造方法，底层靠虚拟机实现，但Externalizable不依赖于虚拟机提供的任何帮助，要求类显式的提供无参构造方法，否则不能序列化    
 

  
           
```
     
    import java.io.*;

    /**
     * Created by elaine on 9/5/15.
     */
     public class StringList implements Externalizable {
        private static final Long serialVersionUID = 1L;

        transient private int size = 0;
        transient private Entry head = null;

    private static class Entry implements Serializable {
        private static final long serialVersionUID = 1L;
        String data;
        Entry next;
        Entry pre;
    }

    public StringList(String headString) {
        initHead(headString);
    }

    private void initHead(String headString) {
        this.head = new Entry();
        this.head.data = headString;
        this.head.next = this.head;
        this.head.pre = this.head;
        this.size = 1;
    }

    public StringList add(String str) {
        Entry newEntry = new Entry();
        newEntry.data = str;
        newEntry.pre = this.head.pre;
        this.head.pre.next = newEntry;
        this.head.pre = newEntry;
        newEntry.next = this.head;

        this.size ++;
        return this;
    }

    public String toString() {
        Entry e = head;
        StringBuilder sb = new StringBuilder(e.data);
        while (hasNext(e)) {
            e = e.next;
            sb.append("->" + e.data);
        }
        return sb.toString();
    }

    private boolean hasNext(Entry e) {
        return e.next != this.head;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeInt(size);
        Entry e = head;
        out.writeUTF(e.data);
        
        while(hasNext(e)) {
            e = e.next;
            out.writeUTF(e.data);
        }
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        int size = in.readInt();
        String headStr = in.readUTF();
        initHead(headStr);
        for (int i = 1; i < size; i++) {
            String str = in.readUTF();
            add(str);
        }
    }

    //无参构造方法
    public StringList() {
        System.out.println("无参构造方法");
        initHead("");
    }

    public static void main(String[] arg) throws IOException, ClassNotFoundException{
        StringList sl = new StringList("yuanqi");
        sl.add("is").add("Elaine");
        System.out.println(sl);
        SerialUtils.writeObject(sl, "sl.byte");
        SerialUtils.readObject("sl.byte");
    }   
}  
```
    
   


   


    