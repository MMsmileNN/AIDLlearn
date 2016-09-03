# AIDLlearn
## AIDL的使用步骤：

###第一：在需要引用的工程中创建AIDL的包存储.aidl文件，并生成.java文件在studio中编译一下就可生成，并创建一个服务继承Service类会实现一个 IBinder
			 private IBinder iBinder =new IMyAidlInterface.Stub(){实现的是aidl接口中的方法}
		通过public IBinder onBind(Intent intent) {
        return iBinder;
   		 }返回出去
###第二：在使用端拷贝被引用工程的aidl的文件夹及包名，编译也会生成一个java文件。然后再activity中去绑定被引用工程的service
####1.启动service
 	 	Intent intent=new Intent();
        //android5.0以后  intent要显示启动 ，ComponentName两个参数是包名，类名（类名要是全路径）
        intent.setComponent(new ComponentName("com.example.zhou.aidl","com.example.zhou.aidl.IMyAidlInterface"));
        bindService(intent,conn, Context.BIND_AUTO_CREATE);//设置类型为创建的时候自动绑定
####2. //获得连接
		    private  ServiceConnection conn=new ServiceConnection() {
		        @Override
		        //绑定上服务的时候调用该方法
		        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
		            //获得接口对象
		            Log.e("iBinder",iBinder+"");
		            	myAidl=IMyAidlInterface.Stub.asInterface(iBinder);
		        }
		
		        @Override
		        //当服务断开的时候调用
		        public void onServiceDisconnected(ComponentName componentName) {
		            //释放资源
		            myAidl=null;
		        }
		    };
####3.通过获得的aild对象在需要的方法中去调用aidl的方法  去实现进程间的操作
	 try {
            Log.e("myAidl",myAidl+"");
		//使用aidl的方法
            int res=myAidl.add(num1,num2);
            et3.setText(res+"");
        } catch (RemoteException e) {
            e.printStackTrace();
        }
####4 注销绑定
		@Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(conn);
    }

##AIDL数据传递
	1.基本数据类型（除了short）
	2.char序列
	3.list（存储的类型必须是支持的类型），要指定输入（in）,输出（out）例：in List<String> alist
	4.map类型 同上
	5.Parcelable 序列化

###Parcelable 序列化实现传递注意
####1.例如创建一个person对象
	public class Person  implements Parcelable{
    private String name;
    private int age;
	//去序列化中的数据
    public Person(Parcel parcel) {
        //注意这里必须要有先后顺序先去name后去age，保持顺序一致
        this.name=parcel.readString();
        this.age=parcel.readInt();
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    //描述
    @Override
    public int describeContents() {
        return 0;
    }

    //写，相当于把数据写到parcel中打成包
    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeString(name);
        parcel.writeInt(age);
    }
    //必须要实现一个方法去读
    public static final Creator<Person> CREATOR = new Creator<Person>() {
        //取出序列化的数据
        @Override
        public Person createFromParcel(Parcel parcel) {
            return new Person(parcel);
        }

        //创建一个数组
        @Override
        public Person[] newArray(int size) {
            return new Person[size];
        }
    };
	}
####2aidl引用中手动导入Person包名并在aidl统一目录下创建一个Person.aidl文件
	文件内容
	// Person.aidl
	package com.example.zhou.aidl;
	//让aidl找到这个bean类对象
	parcelable Person;

####3客户端引用的时候把同样的aidl文件下的文件给客户端一份，就可以使用

	
![](C:\Users\zhou\Desktop\QQ图片20160903165246.png)
