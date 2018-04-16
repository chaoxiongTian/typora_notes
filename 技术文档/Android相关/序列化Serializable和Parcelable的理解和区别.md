### 序列化Serializable和Parcelable的理解和区别

#### 一、android为什么要序列化？什么是序列化？怎么进行序列化？

#####为什么

> 进行Android开发的时候，无法将对象的引用传给Activitiy或者Fragments，我们需要将这些对象放到一个Intent或者Bundle里面，然后再传递。

##### 是什么

>  表示将一个对象转换成可存储或可传输的状态。序列化后的对象可以在网络上进行传输，也可以存储到本地。

##### 怎么做

> Android中Intent如果要传递类对象，可以通过两种方式实现。
>
> 1. Serializable，要传递的类实现Serializable接口传递对象，
> 2. Parcelable，要传递的类实现Parcelable接口传递对象。
>
> Serializable（Java**自带**）： 
>
> Serializable是序列化的意思，表示将一个对象转换成**可存储**或**可传输**的状态。序列化后的对象可以在网络上进行传输，也可以存储到本地。
>
> Parcelable（android **专用**）：
>
> 除了Serializable之外，使用Parcelable也可以实现相同的效果,不过不同于将对象进行序列化，Parcelable方式的实现原理是将一个完整的对象进行分解,而**分解**后的**每一部分**都是Intent所支持的数据类型，这样也就实现传递对象的功能了。

##### 实现Parcelable的作用

1. 永久性保存对象，保存对象的字节序列到本地文件中；
2. 通过序列化对象在网络中传递对象；
3. 通过序列化在进程间传递对象。

##### 选择序列化方法的原则

1. 在**使用内存**的时候，Parcelable比Serializable性能高，所以推荐使用Parcelable。
2. Serializable在序列化的时候会产生大量的临时变量，从而引起频繁的**GC(垃圾回收)**。
3. Parcelable不能使用在要将**数据存储在磁盘上**的情况，因为Parcelable不能很好的保证数据的持续性在外界有变化的情况下。尽管Serializable效率低点，但此时还是建议使用Serializable 。

#### 二、利用java自带的Serializable 进行序列化的例子

弄一个实体类 Person，利用Java自带的Serializable进行序列化

```java
package com.amqr.serializabletest.entity;
import java.io.Serializable;
/**
 * User: LJM
 * Date&Time: 2016-02-22 & 14:16
 * Describe: Describe Text
 */
public class Person implements Serializable{ //这里继承Serializable
    private static final long serialVersionUID = 7382351359868556980L;
    private String name;
    private int age;
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

使用，MainActivity和SecondActivity结合使用 

MainActivity

```java
package com.amqr.serializabletest;
import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.TextView;
import com.amqr.serializabletest.entity.Person;
/**
 * 进行Android开发的时候，我们都知道不能将对象的引用传给Activities或者Fragments，
 * 我们需要将这些对象放到一个Intent或者Bundle里面，然后再传递。
 *
 *
 * Android中Intent如果要传递类对象，可以通过两种方式实现。
 * 方式一：Serializable，要传递的类实现Serializable接口传递对象，
 * 方式二：Parcelable，要传递的类实现Parcelable接口传递对象。
 *
 * Serializable（Java自带）：
 * Serializable是序列化的意思，表示将一个对象转换成可存储或可传输的状态。序列化后的对象可以在网络上进行传输，也可以存储到本地。
 *
 * Parcelable（android 专用）：
 * 除了Serializable之外，使用Parcelable也可以实现相同的效果，
 * 不过不同于将对象进行序列化，Parcelable方式的实现原理是将一个完整的对象进行分解，
 * 而分解后的每一部分都是Intent所支持的数据类型，这样也就实现传递对象的功能了。
 要求被传递的对象必须实现上述2种接口中的一种才能通过Intent直接传递。
 */
public class MainActivity extends Activity {
    private TextView mTvOpenNew;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findViewById(R.id.mTvOpenNew).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent open = new Intent(MainActivity.this,SecondActivity.class);
                Person person = new Person();
                person.setName("一去二三里");
                person.setAge(18);
                // 传输方式一，intent直接调用putExtra 直接传递一个对象
                // public Intent putExtra(String name, Serializable value)
                open.putExtra("put_ser_test", person);
                // 传输方式二，intent利用putExtras（注意s）传入bundle 把一个对象放在bundle中传递
                /**
                Bundle bundle = new Bundle();
                bundle.putSerializable("bundle_ser",person);
                open.putExtras(bundle);
                 */
                startActivity(open);
            }
        });
    }
}
```

SecondActivity(用来接收)

```java
package com.amqr.serializabletest;
import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.widget.TextView;
import com.amqr.serializabletest.entity.Person;
/**
 * User: LJM
 * Date&Time: 2016-02-22 & 11:56
 * Describe: Describe Text
 */
public class SecondActivity extends Activity{
    private TextView mTvDate;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        mTvDate = (TextView) findViewById(R.id.mTvDate);
        Intent intent = getIntent();
        // 关键方法：getSerializableExtra ，我们的类是实现了Serializable接口的，所以写这个方法获得对象
        // public class Person implements Serializable
        Person per = (Person)intent.getSerializableExtra("put_ser_test");
        //Person per = (Person)intent.getSerializableExtra("bundle_ser");
        mTvDate.setText("名字："+per.getName()+"\\n"
                +"年龄："+per.getAge());
    }
}
```

#### 三、android专用的Parcelable的序列化的例子

我们写一个实体类，实现Parcelable接口，马上就被要求

1、复写**describeContents方法**和**writeToParcel方法**

2、实例化**静态内部对象CREATOR**，实现**接口Parcelable.Creator** 。

也就是，随便一个类实现了Parcelable接口就一开始就会变成这样子

Parcelable方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这样也就实现传递对象的功能了。

```java
public class Pen implements Parcelable{
    private String color;
    private int size;
  // 系统自动添加，给createFromParcel里面用
    protected Pen(Parcel in) {
        color = in.readString();
        size = in.readInt();
    }
    public static final Creator<Pen> CREATOR = new Creator<Pen>() {
      @Override
        public Pen createFromParcel(Parcel in) {
            return new Pen(in);// 在构造函数里面完成了 读取 的工作
        }
       //供反序列化本类数组时调用的  
      @Override
        public Pen[] newArray(int size) {
            return new Pen[size];
        }
    };
    @Override
    public int describeContents() {
        return 0;// 内容接口描述，默认返回0即可。
    }
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(color); // 写出 color
        dest.writeInt(size);// 写出 size
    }
}
```

系统已经帮我们做了很多事情，我们需要做的很简单，就写写我们自己需要的构造方法，写一下私有变量的get和set

```java
package com.amqr.serializabletest.entity;
import android.os.Parcel;
import android.os.Parcelable;
/**
 * User: LJM
 * Date&Time: 2016-02-22 & 14:52
 * Describe: Describe Text
 */
public class Pen implements Parcelable{
    private String color;
    private int size;

    // 系统自动添加，给createFromParcel里面用
    protected Pen(Parcel in) {
        color = in.readString();
        size = in.readInt();
    }
    public static final Creator<Pen> CREATOR = new Creator<Pen>() {
        @Override
        public Pen createFromParcel(Parcel in) {
            return new Pen(in); // 在构造函数里面完成了 读取 的工作
        }
        //供反序列化本类数组时调用的
        @Override
        public Pen[] newArray(int size) {
            return new Pen[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;  // 内容接口描述，默认返回0即可。
    }
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(color);  // 写出 color
        dest.writeInt(size);  // 写出 size
    }
    // ======分割线，写写get和set
    //个人自己添加
    public Pen() {
    }
    //个人自己添加
    public Pen(String color, int size) {
        this.color = color;
        this.size = size;
    }

    public String getColor() {
        return color;
    }
    public void setColor(String color) {
        this.color = color;
    }
    public int getSize() {
        return size;
    }
    public void setSize(int size) {
        this.size = size;
    }
}
```

其实说起来Parcelable写起来也不是很麻烦，在as里面，我们的一个实体类写好私有变量之后，让这个类继承自Parcelable，接下的步骤是：

1. 复写两个方法，分别是describeContents和writeToParcel
2. 实例化静态内部对象CREATOR，实现接口Parcelable.Creator 。 以上这两步系统都已经帮我们自动做好了
3. 自己写写我们所需要的构造方法，变量的get和set

实现自Parcelable实体Bean已经写好了，接下来我们结合MainActivity和ThirdActivity来使用以下：

MainActivity

```java
package com.amqr.serializabletest;
import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import com.amqr.serializabletest.entity.Pen;
import com.amqr.serializabletest.entity.Person;
public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // 采用Parcelable的方式
        findViewById(R.id.mTvOpenThird).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent mTvOpenThird = new Intent(MainActivity.this,ThirdActivity.class);
                Pen tranPen = new Pen();
                tranPen.setColor("big red");
                tranPen.setSize(98);
                // public Intent putExtra(String name, Parcelable value)
                mTvOpenThird.putExtra("parcel_test",tranPen);
                startActivity(mTvOpenThird);
            }
        });
    }
}
```

ThirdActivity

```java
package com.amqr.serializabletest;
import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;
import com.amqr.serializabletest.entity.Pen;
/**
 * User: LJM
 * Date&Time: 2016-02-22 & 14:47
 * Describe: Describe Text
 */
public class ThirdActivity extends Activity{
    private TextView mTvThirdDate;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_third);
        mTvThirdDate = (TextView) findViewById(R.id.mTvThirdDate);
//        Intent intent = getIntent();
//        Pen pen = (Pen)intent.getParcelableExtra("parcel_test");
        Pen pen = (Pen)getIntent().getParcelableExtra("parcel_test");
        mTvThirdDate = (TextView) findViewById(R.id.mTvThirdDate);
        mTvThirdDate.setText("颜色:"+pen.getColor()+"\\n"
                            +"大小:"+pen.getSize());
    }
}
```

#### 四、Serializable 和Parcelable的对比

android上应该尽量采用Parcelable，效率至上

**编码上：**

Serializable代码量少，写起来方便

Parcelable代码多一些

**效率上：**

Parcelable的速度比Serializable 高十倍以上

**serializable的迷人之处在于你只需要对某个类以及它的属性实现Serializable 接口即可。Serializable 接口是一种标识接口（marker interface），这意味着无需实现方法，Java便会对这个对象进行高效的序列化操作。这种方法的缺点是使用了反射，序列化的过程较慢。这种机制会在序列化的时候创建许多的临时对象，容易触发垃圾回收。**

**Parcelable方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这样也就实现传递对象的功能了**

[转自](https://blog.csdn.net/Double2hao/article/details/70145747)