**相关文档：**

- [https://github.com/googlesamples/easypermissions](https://github.com/googlesamples/easypermissions)
- [https://developer.android.com/training/permissions/requesting.html](https://developer.android.com/training/permissions/requesting.html)
- [https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous](https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous)

#使用#
##库的引入##
    dependencies {
  		compile 'pub.devrel:easypermissions:0.1.9'
	}

##具体使用##
###1 检查权限###
    String[] perms = {Manifest.permission.CAMERA, Manifest.permission.CHANGE_WIFI_STATE};
	if (EasyPermissions.hasPermissions(this, perms)) {
   		//...     
	} else {
    	//...
	}

###2 申请权限###
    EasyPermissions.requestPermissions(this, "拍照需要摄像头权限",RC_CAMERA_AND_WIFI, perms);

###3 实现EasyPermissions.PermissionCallbacks接口，直接处理权限是否成功申请###
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        // Forward results to EasyPermissions
        EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults, this);
    }

    //成功
    @Override
    public void onPermissionsGranted(int requestCode, List<String> list) {
        // Some permissions have been granted
        // ...
    }

    //失败
    @Override
    public void onPermissionsDenied(int requestCode, List<String> list) {
        // Some permissions have been denied
        // ...
    }

#特点#
Easypermissions主要简化了对权限申请结果的处理和判断，直接以接口的方式回调处理结果。不需要再自行进行处理。