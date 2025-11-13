# FAQ
### 1.用户空间驱动与设备驱动版本兼容问题
linux 5.10版本集成的1.0.0或1.1.0等低版本panfrost设备驱动。由于用户空间驱动和设备驱动头文件不一致，适配用户层高版本的Mesa3D可能会导致初始化失败。
用户驱动调用panfrost_query_raw查询了设备驱动不支持的feature，如DRM_PANFROST_PARAM_AFBC_FEATURES等。  
可以尝试降级Mesa3D版本，或对Mesa3D执行适当的修改：
```
diff --git a/src/panfrost/lib/kmod/panfrost_kmod.c b/src/panfrost/lib/kmod/panfrost_kmod.c
index d2c91d4442b..6e2f90163db 100644
--- a/src/panfrost/lib/kmod/panfrost_kmod.c
+++ b/src/panfrost/lib/kmod/panfrost_kmod.c
@@ -198,8 +198,8 @@ panfrost_dev_query_props(const struct pan_kmod_dev *dev,
          fd, DRM_PANFROST_PARAM_TEXTURE_FEATURES0 + i, true, 0);
    }
 
-   props->afbc_features =
-      panfrost_query_raw(fd, DRM_PANFROST_PARAM_AFBC_FEATURES, true, 0);
+   props->afbc_features = true;
+      // panfrost_query_raw(fd, DRM_PANFROST_PARAM_AFBC_FEATURES, true, 0);
 
    panfrost_dev_query_thread_props(dev, props);
```
