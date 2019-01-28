# Android面试题：Intent

---

## Intent传递数据时，可以传递哪些类型数据？

Intent可以传递的数据类型非常的丰富，java的基本数据类型和String以及他们的数组形式都可以，除此之外还可以传递实现了Serializable和Parcelable接口的对象。

## Serializable和Parcelable的区别

- 在使用内存的时候，Parcelable 类比Serializable性能高，所以推荐使用Parcelable类。
- Serializable在序列化的时候会产生大量的临时变量，从而引起频繁的GC。
- Parcelable不能使用在要将数据存储在磁盘上的情况。尽管Serializable效率低点，但在这种情况下，还是建议你用Serializable 。 

实现：

1. Serializable 的实现，只需要继承Serializable 即可。这只是给对象打了一个标记，系统会自动将其序列化。

2. Parcelabel 的实现，需要在类中添加一个静态成员变量 CREATOR，这个变量需要继承 Parcelable.Creator 接口。

``` java
public class MyParcelable implements Parcelable {
    private int mData;
    public int describeContents() {
        return 0;
    }
    public void writeToParcel(Parcel out, int flags) {
        out.writeInt(mData);
    }
    public static final Parcelable.Creator<MyParcelable> CREATOR
            = new Parcelable.Creator<MyParcelable>() {
        public MyParcelable createFromParcel(Parcel in) {
            return new MyParcelable(in);
        }
        public MyParcelable[] newArray(int size) {
            return new MyParcelable[size];
        }
    };
    private MyParcelable(Parcel in) {
        mData = in.readInt();
    }
}
```

## 请描述一下Intent和IntentFilter

Android 中通过 Intent 对象来表示一条消息，一个 Intent对象不仅包含有这个消息的目的地，还可以包含消息的内容，这好比一封 Email，其中不仅应该包含收件地址，还可以包含具体的内容。对于一个 Intent 对象，消息“目的地”是必须的，而内容则是可选项。

通过Intent 可以实现各种系统组件的调用与激活。

IntentFilter: 可以理解为邮局或者是一个信笺的分拣系统， 这个分拣系统通过3个参数来识别：

- Action: 动作view
- Data: 数据uri
- Category : 而外的附加信息

#### Action 匹配

Action 是一个用户定义的字符串，用于描述一个 Android 应用程序组件，一个 IntentFilter 可以包含多个 Action。在 AndroidManifest.xml 的 Activity 定义时可以在其 &lt;intent-filter >节点指定一个 Action 列表用于标示 Activity 所能接受的“动作”。

例如：

``` xml      
<intent-filter>       
    <action android:name="android.intent.action.MAIN"/>       
    <actionandroid:name="cn.itheima.action" />       
    ……       
</intent-filter>
```

如果我们在启动一个 Activity 时使用这样的Intent 对象：

``` java
Intentintent = new Intent();       
intent.setAction("cn.itheima.action");
```

那么所有的 Action 列表中包含了“cn.itheima”的 Activity 都将会匹配成功。Android 预定义了一系列的 Action 分别表示特定的系统动作。这些Action 通过常量的方式定义在 android.content. Intent中，以“ACTION_”开头。我们可以在 Android 提供的文档中找到它们的详细说明。

#### URI 数据匹配

一个 Intent 可以通过 URI 携带外部数据给目标组件。在 &lt;intent-filter >节点中，通过 &lt;data/>节点匹配外部数据。

mimeType 属性指定携带外部数据的数据类型，scheme 指定协议，host、port、path 指定数据的位置、端口、和路径。如下：

``` xml
<data android:mimeType="mimeType"
    android:scheme="scheme"
    android:host="host"
    android:port="port" 
    android:path="path"/>       
```
电话的uri：`tel:12345`

自己定义的uri：`itcast://cn.itcast/person/10`

如果在 Intent Filter 中指定了这些属性，那么只有所有的属性都匹配成功时 URI 数据匹配才会成功。

#### Category 类别匹配

&lt;intent-filter>节点中可以为组件定义一个 Category 类别列表，当 Intent 中包含这个列表的所有项目时 Category 类别匹配才会成功。