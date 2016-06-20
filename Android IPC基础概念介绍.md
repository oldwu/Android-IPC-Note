# Android IPC基础


## 对象序列化

通过Serializable接口和Parcelable接口序列化的对象，可以使用Intent和Binder传递

### Serializable接口

* 编写的类实现Serialzable接口
* 手动或由IDE生成一个ID  

    ``` private static final long serialVersionUID = 1L```

* 进行对象的序列化和反序列化

```java
//序列化
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(path));
out.writeObject(user);
out.close();
//反序列化
ObjectInputStream in = new ObjectInputStream(new FileInputStream(path));
User user1 = (User) in.readObject();
Log.d("Serial", user1.getUserName());
```
### Parcelable接口
例子  
```java
/**
 * Created by wzy on 2016/6/15.
 */
public class User2 implements Parcelable{
    private int UserID;
    private String userName;
    private boolean isMale;
    private Book book;

    public User2(int userID, String userName, boolean isMale) {
        UserID = userID;
        this.userName = userName;
        this.isMale = isMale;
    }

    //反序列化
    protected User2(Parcel in) {
        UserID = in.readInt();
        userName = in.readString();
        isMale = in.readByte() != 0;
        book = in.readParcelable(Book.class.getClassLoader());
    }

    public static final Creator<User2> CREATOR = new Creator<User2>() {
        @Override
        public User2 createFromParcel(Parcel in) {
            return new User2(in);
        }

        @Override
        public User2[] newArray(int size) {
            return new User2[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    /**
     * 序列化功能
     * @param dest
     * @param flags
     */
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(UserID);
        dest.writeString(userName);
        dest.writeByte((byte) (isMale ? 1 : 0));
        dest.writeParcelable(book, flags);
    }
}

```

### Serializable与Parcelable区别
* Serializable开销较大，Parcelable效率更高
* Parcelable在网络传输中更为复杂，Serializable更方便一些


## Binder

Binder是Android的一个类，实现了IBinder接口  
* 从IPC角度来说，Binder是一种虚拟的物理设备，设备驱动是/dev/binder
* 从Android Framework角度来说，Binder是连接各种Manager和相应ManagerService的桥梁
* 从Android应用层来说，Binder是客户端和服务端进行通信的媒介，bindservice时，服务端会返回包含服务端业务调用的Binder对象

Binder主要用在service中，包括AIDL和Messager


