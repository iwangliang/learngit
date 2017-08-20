# [Training](https://developer.android.com/training/index.html)
## [Building Your First App](https://developer.android.com/training/basics/firstapp/index.html)
### Create an Android Project
1. 下载[Android Studio](https://developer.android.com/studio/index.html)
2. create New Project 一路的next

### Run Your App
#### Run on a real device
1. 连接USB，打开开发者选项
2. run

> note: Android 4.2 及以上版本，开发者选项会隐藏，打开设置 -> 关于设备 点击版本号七次，开发者选项就会打开


#### Run on an emulator
1. 创建 AVD 运行

> note :  Android Virtual Device(AVD) 需要下载System Image  

### Build a Simple User Interface






## [适配不同的设备](https://developer.android.com/training/basics/supporting-devices/index.html)
### 资源适配

	1. 在 res/ 下 添加 <resource type>-b+<language code>[+<country code>]
		eg : values-b+es/   mipmap-b+es+ES/
	2. 然后在不同文件夹下，放对应的资源就行了
		eg :  /values-b+es/strings.xml:
		<resources>
	    <string name="hello_world">¡Hola Mundo!</string>
		</resources>
	3. 然后使用它们
		代码中：
			// Get a string resource from your app's Resources
			String hello = getResources().getString(R.string.hello_world);
			
			// Or supply a string resource to a method that requires a string
			TextView textView = new TextView(this);
			textView.setText(R.string.hello_world);
		资源文件
			<ImageView
			    android:layout_width="wrap_content"
			    android:layout_height="wrap_content"
			    android:src="@mipmap/country_flag" />
		

### 屏幕适配



## [使用Fragment](https://developer.android.com/training/basics/fragments/creating.html)
### 使用布局文件定义Fragment 并加载它

		public class ArticleFragment extends Fragment {
		    @Override
		    public View onCreateView(LayoutInflater inflater, ViewGroup container,
		        Bundle savedInstanceState) {
		        // Inflate the layout for this fragment
		        return inflater.inflate(R.layout.article_view, container, false);
		    }
		}

		article_view:
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		    android:orientation="horizontal"
		    android:layout_width="fill_parent"
		    android:layout_height="fill_parent">
		
		    <fragment android:name="com.example.android.fragments.HeadlinesFragment"
		              android:id="@+id/headlines_fragment"
		              android:layout_weight="1"
		              android:layout_width="0dp"
		              android:layout_height="match_parent" />
		
		    <fragment android:name="com.example.android.fragments.ArticleFragment"
		              android:id="@+id/article_fragment"
		              android:layout_weight="2"
		              android:layout_width="0dp"
		              android:layout_height="match_parent" />
		
		</LinearLayout>

> 1. Fragment 的生命周期会受到外层 Activity 生命周期的影响，比如Activity onPause了，那么Fragment 的onPause()回调将会被调用
> 2. FragmentActivity 是为了兼容 API 11 一下的版本，如果你的最低API 在 11以上，可以直接继承 Activity
> 3. 如果是V7 包，应该继承AppCompatActivity
> 4. XML 写死的Fragment 是不能add,remove等操作的

### 运行时动态加载Fragment
#### FragmentManager 用来运行时管理Fragment

		// 占位的容器
		<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
		    android:id="@+id/fragment_container"
		    android:layout_width="match_parent"
		    android:layout_height="match_parent" />

		public class MainActivity extends FragmentActivity {
		    @Override
		    public void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.news_articles);
		
		        // Check that the activity is using the layout version with
		        // the fragment_container FrameLayout
				// 1. 检查是否存在这个容器，因为如果在平板上，用的是另外一套布局，是没有这个容器的
		        if (findViewById(R.id.fragment_container) != null) {
		
		            // However, if we're being restored from a previous state,
		            // then we don't need to do anything and should return or else
		            // we could end up with overlapping fragments.
					// 2.  注意这是在Activity中，因为所有的东西都交给了Fragment 去处理，所以这里如果有保存的状态信息，在恢复时也应该由Fragment 去恢复，如果Activity 再继续处理的话，又新加了一个Frament
		            if (savedInstanceState != null) {
		                return;
		            }
		
		            // Create a new Fragment to be placed in the activity layout
		            HeadlinesFragment firstFragment = new HeadlinesFragment();
		
		            // In case this activity was started with special instructions from an
		            // Intent, pass the Intent's extras to the fragment as arguments
					// 3. 把参数传递到Fragment
		            firstFragment.setArguments(getIntent().getExtras());
		
		            // Add the fragment to the 'fragment_container' FrameLayout
					// 4. 添加并提交
		            getSupportFragmentManager().beginTransaction()
		                    .add(R.id.fragment_container, firstFragment).commit();
		        }
		    }
		}

> 1. 使用 getSupportFragmentManager 来获取Fragment 的管理器


#### 回退栈

	FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
	
	// Replace whatever is in the fragment_container view with this fragment,
	// and add the transaction to the back stack so the user can navigate back
	transaction.replace(R.id.fragment_container, newFragment);
	transaction.addToBackStack(null);
	
	// Commit the transaction
	transaction.commit();

> 1. transaction.addToBackStack(null); 是将该Frament 加到回退栈中，以便用户点击返回键时，能找到它。当你replace 或者 remove时，使用此方法，Fragment 是stop状态，如果不调用则是 destroy状态。
> 2. 该方法带了个参数，只有你要使用 FragmentManager.BackStackEntry APIs 时才加它，其余时候没什么用

#### Fragment间的通讯