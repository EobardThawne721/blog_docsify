# Android Studio

## 安装Gradle

* macos：https://blog.csdn.net/weixin_42195126/article/details/122502207

* Android Studio配置本地Gradle：https://blog.csdn.net/A1_3_9_7/article/details/147744702



## 开始

创建项目
<img src="Android Studio_images/image-20250526183305859.png" alt="image-20250526183305859" style="zoom:23%;" />
<img src="Android Studio_images/image-20250526183030994.png" alt="image-20250526183305859" style="zoom:25%;" />





修改gradle-wrapper.properties，编辑镜像配置

```properties
#修改镜像地址
#distributionUrl=https\://services.gradle.org/distributions/gradle-8.11.1-bin.zip
distributionUrl=https\://mirrors.cloud.tencent.com/gradle/gradle-8.11.1-bin.zip
```



修改settings.gradle

```properties
# 配置插件的镜像地址
pluginManagement {
    repositories {
    
//        google {
//            content {
//                includeGroupByRegex("com\\.android.*")
//                includeGroupByRegex("com\\.google.*")
//                includeGroupByRegex("androidx.*")
//            }
//        }

        // 阿里云镜像（覆盖 Maven Central、Google、JCenter 等）
        maven { setUrl("https://maven.aliyun.com/repository/public/") }
        maven { setUrl("https://maven.aliyun.com/repository/google/") }
        maven { setUrl("https://maven.aliyun.com/repository/jcenter/") }
        maven { setUrl("https://maven.aliyun.com/repository/gradle-plugin/") }
        // 华为云镜像
        maven { setUrl("https://repo.huaweicloud.com/repository/maven/") }
        // 腾讯云镜像
        maven { setUrl("https://mirrors.cloud.tencent.com/nexus/repository/maven-public/") }
        // 网易镜像
        maven { setUrl("https://mirrors.163.com/maven/repository/maven-public/") }
        mavenCentral()
        gradlePluginPortal()
    }
}

# 项目依赖下载的镜像地址
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        // 阿里云镜像（覆盖 Maven Central、Google、JCenter 等）
        maven { setUrl("https://maven.aliyun.com/repository/public/") }
        maven { setUrl("https://maven.aliyun.com/repository/google/") }
        maven { setUrl("https://maven.aliyun.com/repository/jcenter/") }
        maven { setUrl("https://maven.aliyun.com/repository/gradle-plugin/") }
        // 华为云镜像
        maven { setUrl("https://repo.huaweicloud.com/repository/maven/") }
        // 腾讯云镜像
        maven { setUrl("https://mirrors.cloud.tencent.com/nexus/repository/maven-public/") }
        // 网易镜像
        maven { setUrl("https://mirrors.163.com/maven/repository/maven-public/") }
        google()
        mavenCentral()
    }
}

rootProject.name = "My Application"
include ':app'
```

 

新建MainActivity.java和activity_main.xml文件

> xml文件只能以小写和下划线构成，不然会报错

```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
      	//onCreate()是在Activity第一次被创建时所调用
        super.onCreate(savedInstanceState);
      	//设置对应的布局文件
        setContentView(R.layout.activity_main);
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:layout_editor_absoluteX="29dp"
    tools:layout_editor_absoluteY="16dp">
  
</androidx.constraintlayout.widget.ConstraintLayout>
```

<img src="Android Studio_images/image-20250526183910065.png" alt="image-20250526183910065" style="zoom: 33%;" /> 





编辑AndroidManifest.xml，注册主活动

> ==注意：后续创建的Activity活动都需要注册到这里面，否则不能使用==

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!--
        定义Android应用的全局配置,有且仅有一个application标签
            allowBackup：允许通过Google的备份服务备份和恢复应用数据
            dataExtractionRules：定义如何从应用中提取数据进行备份的规则
            fullBackupContent：定义了应用的完整备份规则
            label：应用的名称，通过导入文件里面的方式
            theme:设置应用的主题样式，引用res/values/styles.xml 中定义的 Theme.MyApplication 样式
            targetApi:api的版本
    -->
    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyApplication"
        tools:targetApi="31">

        <!--
           activity定义一个活动,每个Activity都必须在Manifest文件中声明
                name：指定Activity对应的类名
                exported：该Activity可以被外部应用启动(如需要多活动页面跳转设置该值为true)
                intent-filter:意图过滤器(一个软件不能有两个主活动)
                    action:MAIN为指定应用的主入口点
                    category:LAUNCHER表示应用启动时展示的第一个界面
        -->
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
      
      	
        <!--        <activity-->
        <!--            android:name="Activity的文件"-->
        <!--            android:exported="true">-->
        <!--        </activity>-->
    </application>

</manifest>
```





运行MainActivity文件

<img src="./Android Studio_images/image-20250526184153397.png" alt="image-20250526184153397" style="zoom:25%;" /> 





## 相关Api

### 组件查找

* findViewById(R.id.组件id)：在整个全局视图中获取组件，必须先调用 `setContentView()`，否则会返回 null
* view.findViewById(R.id.组件id)：在某个局部视图获取组件，eg：Fragment、Adapter



eg：获取activity_main.xml的组件，并重新设置值

```xml
 <!--activity_main.xml-->	
<TextView
        android:id="@+id/textView1"
        android:layout_width="130dp"
        android:layout_height="0dp"
        android:layout_marginStart="88dp"
        android:layout_marginTop="60dp"
        android:text="这是一个显示的控件"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
```

```java
public class MainActivity extends AppCompatActivity {
  
  	//用于绑定布局文件的控件
  	private TextView textView;
  
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
      	//onCreate()是在Activity第一次被创建时所调用
        super.onCreate(savedInstanceState);
      	//设置对应的布局文件
        setContentView(R.layout.activity_main);
      
       //获取页面的控件id并绑定
        textView = findViewById(R.id.textView1);
       //获取控件元素的文本
       	String data = textView.getText().toString();
      //重新设置文本元素的值
        textView.setText(data + ":123");
    }
}
```

<img src="./Android Studio_images/image-20250527103243143.png" alt="image-20250527103243143" style="zoom: 50%;" /> 





### 点击事件

* 配置onClick事件

```xml
 <!--activity_main.xml里面加入一个按钮-->	
<Button
        android:id="@+id/btn1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:onClick="bigger"
        android:text="+"
        app:layout_constraintStart_toStartOf="@+id/textView1"
        app:layout_constraintTop_toBottomOf="@+id/textView1" />
```

```java
public class MainActivity extends AppCompatActivity {
  
  	//用于绑定布局文件的控件
  	private TextView textView;
  
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
      	//onCreate()是在Activity第一次被创建时所调用
        super.onCreate(savedInstanceState);
      	//设置对应的布局文件
        setContentView(R.layout.activity_main);
      
       //获取页面的控件id并绑定
        textView = findViewById(R.id.textView1);
    }
  
  	//通过在xml文件中配置common attribute的onClick事件：绑定页面中的按钮方法（设置页面属性值）
    public void bigger(View view) {
        textView.setTextSize(23);
    }
}
```

<img src="./Android Studio_images/image-20250527103830840.png" alt="image-20250527103830840" style="zoom:50%;" /><img src="./Android Studio_images/image-20250527103841907.png" alt="image-20250527103841907" style="zoom:50%;" />



* lambda表达式（推荐）

```java
public class MainActivity extends AppCompatActivity {
  
  	//用于绑定布局文件的控件
  	private TextView textView;
    private Button btn;
  
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
      	//onCreate()是在Activity第一次被创建时所调用
        super.onCreate(savedInstanceState);
      	//设置对应的布局文件
        setContentView(R.layout.activity_main);
      
       //获取页面的控件id并绑定
        textView = findViewById(R.id.textView1);
      
      //通过lambda表达式的方法配置onClick事件
        btn = findViewById(R.id.btn1);
        btn.setOnClickListener(view -> {
         		textView.setTextSize(23);
        });
    }
}
```



### 提示框

#### AlertDialog提示框

* AlertDialog.Builder builder = new AlertDialog.Builder(Context)：根据上下文创建居中提示框，建造者模式
  *  setTitle(CharSequence text)：设置标题
  * setMessage(CharSequence text)：设置正文
  * setPositiveButton(CharSequence text, DialogInterface.OnClickListener listener)：设置确认按钮及其事件，如果事件为null表示使用系统默认的事件监听
  * setNegativeButton(CharSequence text, DialogInterface.OnClickListener listener)：设置取消按钮及其事件，如果事件为null表示使用系统默认的事件监听
  * setCancelable(boolean cancelable)：设置是否可被外部取消（点击对话框外部即可关闭对话框,默认true）
  * show()：显示对话框

```java
  AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
  builder.setTitle("标题")
          .setMessage("提示主题")
          //点击确认的消息监听
          .setPositiveButton("ok", (dialog, which) -> {
          })
          //点击取消的消息监听
          .setNegativeButton("cancel", (dialog, which) -> {
          })
          .setCancelable(false)
          .show();
```

<img src="./Android Studio_images/image-20250527105735819.png" alt="image-20250527105735819" style="zoom:50%;" /> 



#### Toast提示框

```java
//提示信息，会在用户屏幕中下方出现提示词 
public static Toast makeText(Context context, CharSequence text, @Duration int duration) {
      return makeText(context, null, text, duration);
  }
```

* Activity当前上下文，即需要在哪个activity中提示
* 提示的内容
* 展示的时间：Toast.LENGTH_SHORT（短时间）、Toast.LENGTH_LONG（长时间）

```java
 Toast.makeText(MainActivity.this, "2025/5/29", Toast.LENGTH_SHORT).show();
```

<img src="./Android Studio_images/image-20250527110353293.png" alt="image-20250527110353293" style="zoom:50%;" /> 





## 控件元素

### 日历控件

```xml
  <CalendarView
        android:id="@+id/calendarView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="71dp"
        android:layout_marginTop="24dp"
        android:layout_marginEnd="71dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"/>
```

```java
public class MainActivity extends AppCompatActivity {
  
   	private CalendarView calendarView;
  
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
      	//onCreate()是在Activity第一次被创建时所调用
        super.onCreate(savedInstanceState);
      	//设置对应的布局文件
        setContentView(R.layout.activity_main);
      
        //日历控件的使用
        calendarView = findViewById(R.id.calendarView1);
      	//日期切换事件
        calendarView.setOnDateChangeListener((view, year, month, dayOfMonth) -> {
            String result = year + "/" + (month + 1) + "/" + dayOfMonth;
            Toast.makeText(MainActivity.this, result, Toast.LENGTH_SHORT).show();
        });
    }
}
```

<img src="./Android Studio_images/image-20250527104716763.png" alt="image-20250527104716763" style="zoom:50%;" /> 





## Intent意图

### 活动跳转

* public Intent(Context packageContext, Class<?> cls)：创建意图
* startActivity(Intent intent)：开启活动，需传入意图对象

```java
//eg: 让当前MainActivity主活动跳转到Login活动
startActivity(new Intent(MainActivity.this, Login.class));
```



#### 数据传递

* public  Intent putExtra(String key,Object value)

```java
//MainActivity活动
public class MainActivity extends AppCompatActivity {
    private Button btn;
  
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
      	//onCreate()是在Activity第一次被创建时所调用
        super.onCreate(savedInstanceState);
      	//设置对应的布局文件
        setContentView(R.layout.activity_main);
  
      //通过lambda表达式的方法配置onClick事件
        btn = findViewById(R.id.btn1);
        btn.setOnClickListener(view -> {
         		//eg: 让当前MainActivity主活动跳转到Login活动
          Intent intent = new Intent(MainActivity.this, LoginActivity.class);
          //传递k为name的v为admin的数据
          intent.putExtra("data", "admin");
          startActivity(intent);
        });
    }
}

//Login活动
public class LoginActivity extends AppCompatActivity  {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.login_activity);
        //获取上一个活动传递的数据
        Intent intent =getIntent();
        String data =intent.getStringExtra("data");
        Toast.makeText(this,data,Toast.LENGTH_SHORT).show();
    }
}
```

<img src="./Android Studio_images/image-20250527111942533.png" alt="image-20250527111942533" style="zoom:50%;" /> 



### 打开网页

```java
findViewById(R.id.btn).setOnClickListener(v -> {
      //设置需要打开的网址：Uri.parse()方法把 网址字符串 解析成 Uri对象
      Uri uri = Uri.parse("http://www.baidu.com");
      //设置系统内置的Action，通过ACTION_VIEW可以和浏览器进行匹配
      Intent intent = new Intent(Intent.ACTION_VIEW, uri);
      startActivity(intent);
  });
```

<img src="./Android Studio_images/image-20250527144952109.png" alt="image-20250527144952109" style="zoom: 50%;" /><img src="./Android Studio_images/image-20250527144915999.png" alt="image-20250527144915999" style="zoom: 20%;" /> 







## Fragment

> 可以嵌套在Activity中的UI模块,多个Fragment可以组合在一个Activity 中，适合做界面拆分与重用。

### 静态Fragment

1. 主活动

```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //不使用系统默认的标题栏
        if (getSupportActionBar() != null) {
            getSupportActionBar().hide();
        }
        setContentView(R.layout.activity_main);
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!--主活动导入标题fragment-->
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/id_fragment_title"
        android:name="com.eobard.myapplication.static_fragment.TitleFragment"
        android:layout_width="match_parent"
        android:layout_height="45dp" />

    <!--主活动导入中间内容fragment-->
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/id_fragment_content"
        android:name="com.eobard.myapplication.static_fragment.ContentFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/id_fragment_title" />
</RelativeLayout>
```

2. 内容Fragment

```java
public class ContentFragment extends Fragment {
    //当Fragment显示页面时调用此方法，返回显示的View
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_content, container, false);
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="使用Fragment做主面板"
        android:textSize="20sp"
        android:textStyle="bold" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

3. 标题Fragment

```java
public class TitleFragment extends Fragment {

    private ImageButton imageButton;

    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        //将xml的自定义布局转换成View对象
        View view = inflater.inflate(R.layout.fragment_title, container, false);

        //在Fragment需要使用view.xxx才来获取相关api方法
        imageButton = view.findViewById(R.id.btn);
        imageButton.setOnClickListener(v -> 
             Toast.makeText(getActivity(), "!!!",Toast.LENGTH_LONG).show());
        return view;
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="45dp"
    android:background="#000000">

    <ImageButton
        android:id="@+id/btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@mipmap/ic_launcher"/>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="标题"
        android:textColor="#fff"
        android:textSize="20sp"
        android:textStyle="bold" />

</RelativeLayout>
```

<img src="./Android Studio_images/image-20250527142832595.png" alt="image-20250527142832595" style="zoom: 33%;" /> 





### 动态Fragment

* FragmentManager：主要用于在Activity中操作Fragment
* FragmentTransaction：保证一些Fragment操作的原子性
  * beginTransaction：开启事务
  * add：往Activity中添加一个Fragment,并在内部根据Add进去的先后顺序形成一个链表
  * remove：从Activity中移除一个Fragment；如果这个Fragment没有添加到回退栈，这个实例会被销毁。
  * replace：先remove后add
  * show：显示之前隐藏的fragment
  * hide：隐藏当前的fragment，设为不可见并且不销毁；如果这个Fragment没有添加到回退栈，不会销毁这个实例
  * commit：提交事务

1. 主活动

```java
public class MainActivity extends AppCompatActivity {
    private Button btnMine, btnFriend;
  	//“我的”片段、“朋友”片段
    private Fragment mineFragment, friendFragment;
  	// 当前显示的fragment
    private Fragment currentActiveFragment; 

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.dynamic_activity_main);

        btnMine = findViewById(R.id.mine);
        btnFriend = findViewById(R.id.friend);

        //先初始化两个fragment
        mineFragment = new MineFragment();
        friendFragment = new FriendFragment();

        //页面初始化时默认显示“我的”页面
        getSupportFragmentManager().beginTransaction()
                //设置添加的Fragment的对象，指定自定义名称，可以根据FragmentManager.findFragmentByTag获取
                .add(R.id.fragment_container, mineFragment, "mine")
                .add(R.id.fragment_container, friendFragment, "friend")
                .hide(friendFragment).commit(); //默认隐藏friendFragment

        //默认激活mineFragment
        currentActiveFragment = mineFragment;

        //根据点击事件动态切换对应的页面fragment
        btnMine.setOnClickListener(v -> loadFragment(mineFragment));
        btnFriend.setOnClickListener(v -> loadFragment(friendFragment));
    }

    private void loadFragment(Fragment targetFragment) {
        //如果是一样的，说明不用切换
        if (currentActiveFragment == targetFragment) return;
        //否则需要根据传入的新页面进行切换显示
        getSupportFragmentManager().beginTransaction()
                .show(targetFragment)           //显示当前激活的targetFragment
                .hide(currentActiveFragment)    //隐藏上一个
                .commit();
        //将currentActiveFragment变为targetFragment
        currentActiveFragment = targetFragment;
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

   	<!--Fragment容器：动态加载需要显示变化的Fragment-->
    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginTop="90dp"
        android:layout_weight="1" />

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <Button
            android:id="@+id/mine"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:text="我的"
            android:layout_height="wrap_content" />

        <Button
            android:id="@+id/friend"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:text="朋友"
            android:layout_height="wrap_content" />
    </LinearLayout>
</LinearLayout>
```

2. 我的片段

```java
//fragment可以不用在AndroidManifest.xml中注册、也可以不用写对应的xml布局文件
public class MineFragment extends Fragment {

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        TextView textView = new TextView(getContext());
        textView.setText("这是我的页面");
        textView.setTextSize(20);
        textView.setPadding(40, 40, 40, 40);
        return textView;
    }
}
```

3. 朋友片段

```java
public class FriendFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        TextView view = new TextView(getContext());
        view.setText("这是朋友页面");
        view.setTextSize(20);
        view.setPadding(40, 40, 40, 40);
        return view;
    }
}
```

<img src="./Android Studio_images/image-20250527144536093.png" alt="image-20250527144536093" style="zoom: 33%;" /> <img src="./Android Studio_images/image-20250527144554846.png" alt="image-20250527144554846" style="zoom:33%;" />









## Adapter适配器

> ListView和Spinner等都是容器，需要使用单独的某一项布局文件去填充这个容器，所以会存在自定义每项的布局文件，可以使用TextView去充当每行的布局



### ArrayAdapter

> 最简单的适配器，数据源为文本字符串数组

* ArrayAdapter(context,resource每项的布局文件,数据源集合)

```java
public class FriendFragment extends Fragment {
    private final String[] GAMES = {"英雄联盟", "守望先锋", "DOTA2", "CS2", "三角洲", "穿越火线", "地下城与勇士"};

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        //将xml的布局转换成View对象
        View view = inflater.inflate(R.layout.friend, container, false);
        //下拉框
        Spinner spinner = view.findViewById(R.id.spinner2);

        ArrayAdapter<String> adapter = new ArrayAdapter<>(
                getContext(),
                R.layout.drop_down_item, // 自定义的每项布局文件
                GAMES
        );
        spinner.setAdapter(adapter);

        //下拉框选择的事件
        spinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                //显示提示框
                String selectedItem = GAMES[position];
                Toast.makeText(getContext(), "选中的是：" + selectedItem, Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {
            }
        });
        return view;
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Spinner
        android:id="@+id/spinner2"
        android:layout_width="409dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="10dp"
        android:layout_marginTop="38dp"
        android:layout_marginEnd="10dp"
       />

</androidx.constraintlayout.widget.ConstraintLayout>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="8dp"
    android:textSize="16sp" />
```

<img src="./Android Studio_images/image-20250527150227832.png" alt="image-20250527150227832" style="zoom: 33%;" /> 







### SimpleAdapter

> 简单适配器，数据源结构比较复杂，一般为`List<Map>`类型对象

* SimpleAdapter(context, List数据源集合, resource每项的布局文件, 数据源中需要展示的k, 每项布局文件中对应k的id值)

```xml
<!--布局文件-->
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@+id/listView1"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_margin="10dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

```java
public class Second extends Activity {
  
    private ListView listView;

    //数据源
    public static ArrayList<Map<String,Object>> arrayList = new ArrayList<>() ;

    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.second);

        fillData();
        listView=findViewById(R.id.listView1);
      
        SimpleAdapter adapter = new SimpleAdapter(
                Second.this,    							//在哪个上下文中显示
                arrayList,              			//数据源
          	// 每个列表项的布局文件（ListView是一个容器，需要自定义的每项的单独布局文件）
                R.layout.list_item,     			
                new String[]{"name", "age"},  // 数据源中的要展示的键名（数据源中的字段k）
                new int[]{R.id.nameTextView, R.id.ageTextView});//视图控件中对应的ID
        
      	listView.setAdapter(adapter);
    }

    // 模拟填充数据
    private void fillData() {
        Map<String, Object> item1 = new HashMap<>();
        item1.put("name", "John Doe");
        item1.put("age", "25");

        Map<String, Object> item2 = new HashMap<>();
        item2.put("name", "Jane Smith");
        item2.put("age", "30");

        arrayList.add(item1);
        arrayList.add(item2);
    }
}
```

```xml
<!--每项单独的布局文件-->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="16dp">

    <!-- 显示名字的 TextView -->
    <TextView
        android:id="@+id/nameTextView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Name"
        android:textSize="16sp" />

    <!-- 显示年龄的 TextView -->
    <TextView
        android:id="@+id/ageTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Age"
        android:textSize="16sp" />
</LinearLayout>
```

<img src="./Android Studio_images/image-20250527133831882.png" alt="image-20250527133831882" style="zoom:50%;" /> 



## 数据存储

### SharedPreferences

> 轻量级的数据存储方式，用来保存App的各种配置信息。以xml形式持久化存储到用户本地设备上(直到手动删除、卸载设备或恢复出厂设置)。

```java
public class Login extends Activity {
    private EditText userName;
    private EditText password;
    private CheckBox rememberMe;

    private SharedPreferences sp;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.login);
        //初始化获取页面控件
        initView();

        //获取SharedPreferences对象
        // getSharedPreferences(文件的名称默认以xml结尾，文件的读写权限（私有的，只有当前应用能访问，其他应用无法读取或修改）)
        sp = getSharedPreferences("config", Context.MODE_PRIVATE);
        //获取SharedPreferences对应的k,如果获取不到则设置默认值
        String inputUserName = sp.getString("userName", null);
        String inputPassword = sp.getString("password", null);
        boolean isChecked = sp.getBoolean("isChecked", false);

        //回显数据
        userName.setText(inputUserName);
        password.setText(inputPassword);
        rememberMe.setChecked(isChecked);
    }

    private void initView() {
        userName = findViewById(R.id.userName);
        password = findViewById(R.id.password);
        rememberMe = findViewById(R.id.rememberMe);
    }

  	//登录逻辑
    public void login(View view) {
        String inputUserName = userName.getText().toString().trim();
        String inputPassword = password.getText().toString().trim();
        if (TextUtils.isEmpty(inputPassword) || TextUtils.isEmpty(inputUserName)) {
            Toast.makeText(Login.this, "用户名或密码为空", Toast.LENGTH_SHORT).show();
          	return;
        }
        //省略具体登录业务逻辑

        //如果使用了记住我功能，则保存到SharedPreferences中
        if (rememberMe.isChecked()) {
            saveToSharedPreferences(inputUserName, inputPassword, rememberMe.isChecked());
        } else {
            cleanSharedPreferences();
        }

        Toast.makeText(Login.this, "登录成功", Toast.LENGTH_SHORT).show();

        //延时5秒返回主页
        new Handler().postDelayed(() -> {
            startActivity(new Intent(Login.this, MainActivity.class));
        }, 1000);
    }

  	//保存SharedPreferences信息
    private boolean saveToSharedPreferences(String userName, String password, boolean isChecked) {
        //获得读写操作
        SharedPreferences.Editor edit = sp.edit();
        //设置值
        edit.putString("userName", userName);
        edit.putString("password", password);
        edit.putBoolean("isChecked", isChecked);
        /**
         * commit：同步方法，返回保存结果，保存数据会阻塞当前线程，直到保存操作完成
         * apply：异步方法，不返回保存结果，不阻塞当前线程
         */
        return edit.commit();
    }

    //清除SharedPreferences信息
    private void cleanSharedPreferences() {
        SharedPreferences.Editor editor = sp.edit();
        //editor.clear();   //清除全部
        editor.remove("userName");
        editor.remove("password");
        editor.remove("isChecked");
        editor.apply();  // 异步保存
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp">

    <EditText
        android:id="@+id/userName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="please input your name"
        />

    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/password"
        android:hint="please input your paseword"
        android:layout_marginTop="10dp"/>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="40dp">

        <CheckBox
            android:id="@+id/rememberMe"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerVertical="true"
            android:text="记住我"
            />
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:layout_marginLeft="12dp"
            android:onClick="login"
            android:text="登录" />
    </RelativeLayout>
</LinearLayout>
```

<img src="./Android Studio_images/image-20250527135629729.png" alt="image-20250527135629729" style="zoom:50%;" /> <img src="./Android Studio_images/image-20250527135652350.png" alt="image-20250527135652350" style="zoom:50%;" />

<img src="./Android Studio_images/image-20250527135551773.png" alt="image-20250527135551773" style="zoom:50%;" /> 



### ViewModel

> **多个Fragment/Activity之间可以共享数据，并且屏幕旋转或界面销毁重建时保留数据,避免 Activity/Fragment因生命周期而频繁创建销毁数据**。

eg: 两个Fragment之间数据传递

1.ViewModel

```java
public class SharedViewModel extends ViewModel {

    private final MutableLiveData<Map<String, Object>> sharedData = new MutableLiveData<>();

    //设置共享数据
    public void setSharedData(Map<String,Object> data){
        sharedData.setValue(data);
    }

    //获取共享数据
    public MutableLiveData<Map<String, Object>> getSharedData() {
        return sharedData;
    }
}
```

2. 朋友片段

```java
public class FriendFragment extends Fragment {
  	private SharedViewModel sharedViewModel;
  
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        TextView view = new TextView(getContext());
        view.setText("这是朋友页面");
        view.setTextSize(20);
        view.setPadding(40, 40, 40, 40);
      
      //获取当前Activity作用域下的ViewModel，fragment利用ViewModel进行共享数据
        sharedViewModel = new 	ViewModelProvider(requireActivity()).get(SharedViewModel.class);
      
      	//设置共享数据值
        HashMap<String, Object> data = new HashMap<>();
        data.put("name", "eobard thawne");
        data.put("age", "25");
        data.put("sex", "male");
        sharedViewModel.setSharedData(data);
        return view;
    }
}
```

3. 我的片段

```java
public class MineFragment extends Fragment {
  
    private SharedViewModel sharedViewModel;

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        TextView textView = new TextView(getContext());
        textView.setText("这是我的页面");
        textView.setTextSize(20);
        textView.setPadding(40, 40, 40, 40);

        sharedViewModel = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
      
        sharedViewModel.getSharedData().observe(getViewLifecycleOwner(), map -> {
            Object name = map.get("name");
            Object age = map.get("age");
            Object sex = map.get("sex");
            textView.setText(name + ":" + age + ":" + sex);
        });

        return textView;
    }
}
```

<img src="./Android Studio_images/image-20250527152611360.png" alt="image-20250527152611360" style="zoom:50%;" /> 







## MVP开发模式

> Model-View-Presenter,MVP
>
> ```tex
>  +--------+        +------------+         +--------+
>  |  View  | <----> | Presenter  | <-----> | Model  |
>  +--------+        +------------+         +--------+
> ```

* View层：
  * interface：提供给Presenter的抽象方法，一般是数据响应给用户的抽象业务
  * class：Activity或Fragment 实现View接口，用户的点击事件、实现具体对应的抽象响应业务，eg：如何展示成功/失败提示
* Presenter（interface & class）：根据Model层的结果去操作一个抽象的View而不是具体的Activity，这样可以让 Presenter 和具体的 UI 实现解耦
* Model（interface & class）：数据库访问或API请求、具体的业务逻辑

eg：用户登录

```java
//model层
public interface LoginModel {
    boolean checkUser(String username, String password);
}

public class LoginModelImpl implements LoginModel {
    @Override
    public boolean checkUser(String username, String password) {
        return "admin".equals(username) && "123".equals(password);
    }
}
```

```java
//presenter层
public interface LoginPresenter {
    void login(String username, String password);
}

public class LoginPresenterImpl implements LoginPresenter {
    //引入具体的model层进行数据操作和业务逻辑
    private final LoginModel loginModel;
    //引入抽象的View层需要展现的内容结果
    private final LoginView loginView;

    public LoginPresenterImpl(LoginView loginView) {
       //final修饰的成员变量在构造方法或者声明时赋值，一旦创建就不能被修改
        this.loginModel = new LoginModelImpl();
        this.loginView = loginView;
    }

    @Override
    public void login(String username, String password) {
        //获取model层结果
        boolean status = loginModel.checkUser(username, password);
        //根据结果去渲染对应的逻辑
        if (status) {
            loginView.showLoginSuccess("欢迎回来！");
        } else {
            loginView.showLoginFailure("请检查用户名或密码");
        }
    }
}
```

```java
//view层
public interface LoginView {
    //登录成功展示的逻辑
    void showLoginSuccess(String message);
    //登录失败展示的逻辑
    void showLoginFailure(String message);
}

public class LoginActivity extends AppCompatActivity implements LoginView {

    private EditText username;
    private EditText password;
    private LoginPresenter loginPresenter;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.login_activity);
				
      	//实例化presenter层
        loginPresenter = new LoginPresenterImpl(LoginActivity.this);

        username = findViewById(R.id.username);
        password = findViewById(R.id.password);

      	//返回主界面按钮
        findViewById(R.id.returnHome).setOnClickListener(view -> {
            startActivity(new Intent(LoginActivity.this, MainActivity.class));
        });

      	//登录按钮
        findViewById(R.id.loginMvp).setOnClickListener(view -> {
            loginPresenter.login(username.getText().toString().trim(), password.getText().toString().trim());
        });
    }

		//具体的登录成功需要展示的逻辑
    @Override
    public void showLoginSuccess(String message) {
        Toast.makeText(this, "登录成功: " + message, Toast.LENGTH_SHORT).show();
        new Handler().postDelayed(() -> {
            startActivity(new Intent(LoginActivity.this, MainActivity.class));
        }, 1000);
    }

  	//具体的登录失败需要展示的逻辑
    @Override
    public void showLoginFailure(String message) {
        Toast.makeText(this, "登录失败: " + message, Toast.LENGTH_SHORT).show();
    }
}
```

```xml
<!--login_activity.xml布局文件-->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="10dp">

    <EditText
        android:id="@+id/username"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="40dp"
        android:hint="please input your name" />

    <EditText
        android:id="@+id/password"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:hint="please input your paseword" />


    <Button
        android:id="@+id/loginMvp"
        android:layout_width="392dp"
        android:layout_height="wrap_content"
        android:text="登录" />

    <Button
        android:id="@+id/returnHome"
        android:layout_width="392dp"
        android:layout_height="wrap_content"
        android:text="返回主界面" />

</LinearLayout>
```

<img src="./Android Studio_images/image-20250527141315720.png" alt="image-20250527141315720" style="zoom: 33%;" /><img src="./Android Studio_images/image-20250527141329054.png" alt="image-20250527141329054" style="zoom: 33%;" />
