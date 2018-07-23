**GitHub:[https://github.com/hotchemi/PermissionsDispatcher](https://github.com/hotchemi/PermissionsDispatcher)**

**参考资料：[http://www.cnblogs.com/duduhuo/p/6228426.html](http://www.cnblogs.com/duduhuo/p/6228426.html)**

#使用#
##1、添加依赖##
首先在**项目工程下的build.gradle**文件中加入对maven仓库依赖引入的支持。

    allprojects {
	    repositories {
	        jcenter()
	        mavenCentral()
	    }
	}

之后在**module下的build.gradle**文件中添加两项依赖：

    compile 'com.github.hotchemi:permissionsdispatcher:2.3.1'
	annotationProcessor 'com.github.hotchemi:permissionsdispatcher-processor:2.3.1'

并将targetSdkVersion设为23，即：targetSdkVersion 23

##2、在Activity或Fragment中使用##
**相关注解如下：**
<table>
	<tr>
		<th>
			Annotation
		</th>
		<th>
			Required
		</th>
		<th>
			Annotation
		</th>
	</tr>
	<tr>
		<th>
			@RuntimePermissions
		</th>
		<th>
			✓
		</th>
		<th>
			注解在其内部需要使用运行时权限的Activity或Fragment上
		</th>
	</tr>
	<tr>
		<th>
			@NeedsPermission
		</th>
		<th>
			✓
		</th>
		<th>
			注解在需要调用运行时权限的方法上，当用户给予权限时会执行该方法
		</th>
	</tr>
	<tr>
		<th>
			@OnShowRationale
		</th>
		<th>
			
		</th>
		<th>
			注解在用于向用户解释为什么需要调用该权限的方法上，只有当第一次请求权限被用户拒绝，下次请求权限之前会调用
		</th>
	</tr>
	<tr>
		<th>
			@OnPermissionDenied
		</th>
		<th>
			
		</th>
		<th>
			注解在当用户拒绝了权限请求时需要调用的方法上
		</th>
	</tr>
	<tr>
		<th>
			@OnNeverAskAgain
		</th>
		<th>
			
		</th>
		<th>
			注解在当用户选中了授权窗口中的不再询问复选框后并拒绝了权限请求时需要调用的方法，一般可以向用户解释为何申请此权限，并根据实际需求决定是否再次弹出权限请求对话框
		</th>
	</tr>
</table>
> **注意：被注解的方法不能是私有方法。**

只有**@RuntimePermissions**和**@NeedsPermission**是必须的，其余注解均为可选。当使用了**@RuntimePermissions**和**@NeedsPermission**之后，需要点击菜单栏中Build菜单下的Make Project，或者按快捷键Ctrl + F9编译整个项目，编译器会在**app\build\intermediates\classes\debug**目录下与被注解的Activity同一个包下生成一个辅助类，**名称为被注解的Activity名称+PermissionsDispatcher.class**，如图所示：
![](http://img.blog.csdn.net/20161227201859240)

接下来可以调用辅助类里面的方法完成应用的权限请求了。

在需要调用权限的位置调用辅助类里面的xxxWithCheck方法，xxx是被@NeedsPermission注解的方法名。如：

    MainActivityPermissionsDispatcher.showCameraWithCheck(this);

之后，还需要重写该Activity的**onRequestPermissionsResult()**方法，其方法内调用辅助类的onRequestPermissionsResult()方法，如下：

    @Override
	public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
	    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
	    // NOTE: delegate the permission handling to generated method
	    MainActivityPermissionsDispatcher.onRequestPermissionsResult(this, requestCode, grantResults);
	}

**完整的MainActivity如下所示：**

    @RuntimePermissions
	public class MainActivity extends AppCompatActivity implements View.OnClickListener {
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        findViewById(R.id.button_camera).setOnClickListener(this);
	        findViewById(R.id.button_contacts).setOnClickListener(this);
	    }
	
	    @Override
	    public void onClick(@NonNull View v) {
	        switch (v.getId()) {
	            case R.id.button_camera:
	                // NOTE: delegate the permission handling to generated method
	                MainActivityPermissionsDispatcher.showCameraWithCheck(this);
	                break;
	            case R.id.button_contacts:
	                // NOTE: delegate the permission handling to generated method
	                MainActivityPermissionsDispatcher.showContactsWithCheck(this);
	                break;
	        }
	    }
	
	    @Override
	    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
	        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
	        // NOTE: delegate the permission handling to generated method
	        MainActivityPermissionsDispatcher.onRequestPermissionsResult(this, requestCode, grantResults);
	    }
	
	    @NeedsPermission(Manifest.permission.CAMERA)
	    void showCamera() {
	        // NOTE: Perform action that requires the permission. If this is run by PermissionsDispatcher, the permission will have been granted
	        getSupportFragmentManager().beginTransaction()
	                .replace(R.id.sample_content_fragment, CameraPreviewFragment.newInstance())
	                .addToBackStack("camera")
	                .commitAllowingStateLoss();
	    }
	
	    @NeedsPermission({Manifest.permission.READ_CONTACTS, Manifest.permission.WRITE_CONTACTS})
	    void showContacts() {
	        // NOTE: Perform action that requires the permission.
	        // If this is run by PermissionsDispatcher, the permission will have been granted
	        getSupportFragmentManager().beginTransaction()
	                .replace(R.id.sample_content_fragment, ContactsFragment.newInstance())
	                .addToBackStack("contacts")
	                .commitAllowingStateLoss();
	    }
	
	    @OnShowRationale(Manifest.permission.CAMERA)
	    void showRationaleForCamera(PermissionRequest request) {
	        // NOTE: Show a rationale to explain why the permission is needed, e.g. with a dialog.
	        // Call proceed() or cancel() on the provided PermissionRequest to continue or abort
	        showRationaleDialog(R.string.permission_camera_rationale, request);
	    }
	
	    @OnShowRationale({Manifest.permission.READ_CONTACTS, Manifest.permission.WRITE_CONTACTS})
	    void showRationaleForContact(PermissionRequest request) {
	        // NOTE: Show a rationale to explain why the permission is needed, e.g. with a dialog.
	        // Call proceed() or cancel() on the provided PermissionRequest to continue or abort
	        showRationaleDialog(R.string.permission_contacts_rationale, request);
	    }
	
	    @OnPermissionDenied(Manifest.permission.CAMERA)
	    void onCameraDenied() {
	        // NOTE: Deal with a denied permission, e.g. by showing specific UI
	        // or disabling certain functionality
	        Toast.makeText(this, R.string.permission_camera_denied, Toast.LENGTH_SHORT).show();
	    }
	
	    @OnNeverAskAgain(Manifest.permission.CAMERA)
	    void onCameraNeverAskAgain() {
	        Toast.makeText(this, R.string.permission_camera_never_askagain, Toast.LENGTH_SHORT).show();
	    }
	
	    public void onBackClick(View view) {
	        getSupportFragmentManager().popBackStack();
	    }
	
	    private void showRationaleDialog(@StringRes int messageResId, final PermissionRequest request) {
	        new AlertDialog.Builder(this)
	                .setPositiveButton("允许", new DialogInterface.OnClickListener() {
	                    @Override
	                    public void onClick(@NonNull DialogInterface dialog, int which) {
	                        request.proceed();
	                    }
	                })
	                .setNegativeButton("拒绝", new DialogInterface.OnClickListener() {
	                    @Override
	                    public void onClick(@NonNull DialogInterface dialog, int which) {
	                        request.cancel();
	                    }
	                })
	                .setCancelable(false)
	                .setMessage(messageResId)
	                .show();
	    }
	}