---
title: android-Camera2Basic 解析
date: 2019-01-23 17:08:54
tags:
- Android
categories:
- Android开发笔记
---



# Android Camera2介绍

Android 5.0(API Level 21) 重新设计了 Camera，废弃了之前的 Camera，改用现在的 [Camera2 API](https://developer.android.com/reference/android/hardware/camera2/package-summary)，在Camera2上引入了**Session/Request**的概念，使用的复杂度远超之前的 Camera。

<!--more-->

## Camera2主要类简介

- **CameraManager**：摄像头管理类，用于检测，打开系统摄像头，可以通过`getCameraCharacteristics(cameraId)`获取摄像头的特征  
- **CameraCharacteristics**：相机的特性类，可以得到相机是否支持自动对焦，是否支持闪光灯等特征
- **CameraDevice**：相机设备类
- **CameraCaptureSession**：用于创建预览，拍照的Session类。
- **CaptureRequest**：相机的请求类，可以通过这个类获取一次捕获的请求，可以设置一些参数等。
- **CameraRequest**和**CameraRequest.Builder**：当程序调用`setRepeatingRequest()`方法进行预览时，或调用`capture()`方法进行拍照时，都需要传入**CameraRequest**参数。**CameraRequest**代表了一次捕获请求，用于描述捕获图片的各种参数设置，比如对焦模式、曝光模式……总之，程序需要对照片所做的各种控制，都通过**CameraRequest**参数进行设置。**CameraRequest.Builder**则负责生成**CameraRequest**对象。

## Camera2 API 基本架构

![](https://ws2.sinaimg.cn/large/006tNc79ly1fzhvmxx1tlj30ok0amwf1.jpg)

- APP通过创建一个CaptureRequest向CameraDevice发起Capture请求  
- CameraDevice收到请求后返回对应数据的预览数据  
- 点击拍照后，会从ImageReader中读取数据  
- CaptureRequest代表请求控制的Camera参数, CameraMetadata(CaptureResult)则表示当前返回帧中Camera使用的参数以及当前状态.


## 使用流程
![](https://ws1.sinaimg.cn/large/006tNc79ly1fzhvmk3ykcj30f70kv0uc.jpg)

- **初始化，打开Camera**

    1. 通过
    
        ```java
        if (ContextCompat.checkSelfPermission(getActivity(), Manifest.permission.CAMERA)
                != PackageManager.PERMISSION_GRANTED) {
            requestCameraPermission();
            return;
        }
        ```
        获取权限
    2. 通过   `context.getSystemService(Context.CAMERA_SERVICE)` 获取`CameraManager`
    3. 通过`CameraManager.getCameraCharacteristics(cameraId)`获取相机的摄像头的特性
    
        - `SENSOR_ORIENTATION`：获取摄像头的拍照的方向
        - `LENS_FACING`：获取摄像头的方向。`LENS_FACING_FRONT`是前摄像头，`LENS_FACING_BACK`是后摄像头。
        - 获取FPS的范围
        - 获取大小
    4. 调用`CameraManager .open()`方法在回调中得到`CameraDevice`.  
- **Create Session**
    通过`CameraDevice.createCaptureSession() `在回调中获取`CameraCaptureSession`.
- **Config Session**
    构建`CaptureRequest`, 有三种模式可选 预览/拍照/录像.  
- **Capture**
    拍照数据可以在`ImageReader.OnImageAvailableListener`回调中获取, `CaptureCallback`中则可获取拍照实际的参数和Camera当前状态.
    



# android-Camera2Basic介绍

[android-Camera2Basic](https://github.com/googlesamples/android-Camera2Basic)是谷歌官方给的Camera2的demo，演示了Camrea2 API的基本功能，通过这个Demo，我们可以掌握连接设备，显示预览，以及拍照的基本使用。

## 项目结构


```java
├── .DS_Store
└── camera2basic
    ├── AutoFitTextureView.java
    ├── Camera2BasicFragment.java
    └── CameraActivity.java
```


## 代码详解

### 打开相机，创建预览
#### openCamera
```java
private void openCamera(int width, int height) {
        if (ContextCompat.checkSelfPermission(getActivity(), Manifest.permission.CAMERA)
                != PackageManager.PERMISSION_GRANTED) {
            //请求权限
            requestCameraPermission();
            return;
        }
        //设置相机相关的参数
        setUpCameraOutputs(width, height);
        configureTransform(width, height);
        Activity activity = getActivity();
        CameraManager manager = (CameraManager) activity.getSystemService(Context.CAMERA_SERVICE);
        try {
            if (!mCameraOpenCloseLock.tryAcquire(2500, TimeUnit.MILLISECONDS)) {
                throw new RuntimeException("Time out waiting to lock camera opening.");
            }
            manager.openCamera(mCameraId, mStateCallback, mBackgroundHandler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            throw new RuntimeException("Interrupted while trying to lock camera opening.", e);
        }
    }    
```

##### setUpCameraOutputs(width, height)
        
```java
@SuppressWarnings("SuspiciousNameCombination")
    private void setUpCameraOutputs(int width, int height) {
        Activity activity = getActivity();
        CameraManager manager = (CameraManager) activity.getSystemService(Context.CAMERA_SERVICE);
        try {
            //获取摄像头可用列表
            for (String cameraId : manager.getCameraIdList()) {
                //获取相机的特性
                CameraCharacteristics characteristics
                        = manager.getCameraCharacteristics(cameraId);

                // 不使用前置摄像头
                Integer facing = characteristics.get(CameraCharacteristics.LENS_FACING);
                if (facing != null && facing == CameraCharacteristics.LENS_FACING_FRONT) {
                    continue;
                }

                StreamConfigurationMap map = characteristics.get(
                        CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
                if (map == null) {
                    continue;
                }

                // For still image captures, we use the largest available size.
                Size largest = Collections.max(
                        Arrays.asList(map.getOutputSizes(ImageFormat.JPEG)),
                        new CompareSizesByArea());
                //设置ImageReader接收的图片格式，以及允许接收的最大图片数目
                mImageReader = ImageReader.newInstance(largest.getWidth(), largest.getHeight(),
                        ImageFormat.JPEG, /*maxImages*/2);
                //设置图片存储的监听
                mImageReader.setOnImageAvailableListener(
                        mOnImageAvailableListener, mBackgroundHandler);

                ...
            }        
        
```


#### CameraDevice.StateCallback监听
    
```java
private final CameraDevice.StateCallback mStateCallback = new CameraDevice.StateCallback() {

        @Override
        public void onOpened(@NonNull CameraDevice cameraDevice) {
            // This method is called when the camera is opened.  We start camera preview here.
            mCameraOpenCloseLock.release();
            mCameraDevice = cameraDevice;
            //创建PreviewSession
            createCameraPreviewSession();
        }

        @Override
        public void onDisconnected(@NonNull CameraDevice cameraDevice) {
            mCameraOpenCloseLock.release();
            cameraDevice.close();
            mCameraDevice = null;
        }

        @Override
        public void onError(@NonNull CameraDevice cameraDevice, int error) {
            mCameraOpenCloseLock.release();
            cameraDevice.close();
            mCameraDevice = null;
            Activity activity = getActivity();
            if (null != activity) {
                activity.finish();
            }
        }

    };

```
    
#### createCaptureSession
```java
private void createCameraPreviewSession() {
        try {
            SurfaceTexture texture = mTextureView.getSurfaceTexture();
            assert texture != null;

            // We configure the size of default buffer to be the size of camera preview we want.
            texture.setDefaultBufferSize(mPreviewSize.getWidth(), mPreviewSize.getHeight());

            // This is the output Surface we need to start preview.
            Surface surface = new Surface(texture);

            // We set up a CaptureRequest.Builder with the output Surface.
            mPreviewRequestBuilder
                    = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
            mPreviewRequestBuilder.addTarget(surface);

            // Here, we create a CameraCaptureSession for camera preview.
            mCameraDevice.createCaptureSession(Arrays.asList(surface, mImageReader.getSurface()),
                    new CameraCaptureSession.StateCallback() {

                        @Override
                        public void onConfigured(@NonNull CameraCaptureSession cameraCaptureSession) {
                            // The camera is already closed
                            if (null == mCameraDevice) {
                                return;
                            }

                            // When the session is ready, we start displaying the preview.
                            mCaptureSession = cameraCaptureSession;
                            try {
                                // Auto focus should be continuous for camera preview.
                                mPreviewRequestBuilder.set(CaptureRequest.CONTROL_AF_MODE,
                                        CaptureRequest.CONTROL_AF_MODE_CONTINUOUS_PICTURE);
                                // Flash is automatically enabled when necessary.
                                setAutoFlash(mPreviewRequestBuilder);

                                // Finally, we start displaying the camera preview.
                                mPreviewRequest = mPreviewRequestBuilder.build();
                                mCaptureSession.setRepeatingRequest(mPreviewRequest,
                                        mCaptureCallback, mBackgroundHandler);
                            } catch (CameraAccessException e) {
                                e.printStackTrace();
                            }
                        }

                        @Override
                        public void onConfigureFailed(
                                @NonNull CameraCaptureSession cameraCaptureSession) {
                            showToast("Failed");
                        }
                    }, null
            );
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }
```
        
  项目中有注释，不做过多说明

### 拍照

拍照的过程就是我们向已经打开好的相机获取静态预览帧的过程。

#### 对焦

```java
private void lockFocus() {
        try {
            // This is how to tell the camera to lock focus.
            mPreviewRequestBuilder.set(CaptureRequest.CONTROL_AF_TRIGGER,
                    CameraMetadata.CONTROL_AF_TRIGGER_START);
            // Tell #mCaptureCallback to wait for the lock.
            mState = STATE_WAITING_LOCK;
            mCaptureSession.capture(mPreviewRequestBuilder.build(), mCaptureCallback,
                    mBackgroundHandler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }
```

向`CameraCaptureSession`发送对焦请求，并且对对焦是否成功进行监听，在mCaptureCallback中对回调进行处理。

```java
private CameraCaptureSession.CaptureCallback mCaptureCallback
            = new CameraCaptureSession.CaptureCallback() {

        private void process(CaptureResult result) {
            switch (mState) {
                case STATE_PREVIEW: {
                    // We have nothing to do when the camera preview is working normally.
                    break;
                }
                //等待对焦
                case STATE_WAITING_LOCK: {
                    Integer afState = result.get(CaptureResult.CONTROL_AF_STATE);
                    if (afState == null) {
                        captureStillPicture();
                    } else if (CaptureResult.CONTROL_AF_STATE_FOCUSED_LOCKED == afState ||
                            CaptureResult.CONTROL_AF_STATE_NOT_FOCUSED_LOCKED == afState) {
                        // CONTROL_AE_STATE can be null on some devices
                        Integer aeState = result.get(CaptureResult.CONTROL_AE_STATE);
                        if (aeState == null ||
                                aeState == CaptureResult.CONTROL_AE_STATE_CONVERGED) {
                            mState = STATE_PICTURE_TAKEN;
                            //对焦完成
                            captureStillPicture();
                        } else {
                            runPrecaptureSequence();
                        }
                    }
                    break;
                }
                case STATE_WAITING_PRECAPTURE: {
                    // CONTROL_AE_STATE can be null on some devices
                    Integer aeState = result.get(CaptureResult.CONTROL_AE_STATE);
                    if (aeState == null ||
                            aeState == CaptureResult.CONTROL_AE_STATE_PRECAPTURE ||
                            aeState == CaptureRequest.CONTROL_AE_STATE_FLASH_REQUIRED) {
                        mState = STATE_WAITING_NON_PRECAPTURE;
                    }
                    break;
                }
                case STATE_WAITING_NON_PRECAPTURE: {
                    // CONTROL_AE_STATE can be null on some devices
                    Integer aeState = result.get(CaptureResult.CONTROL_AE_STATE);
                    if (aeState == null || aeState != CaptureResult.CONTROL_AE_STATE_PRECAPTURE) {
                        mState = STATE_PICTURE_TAKEN;
                        captureStillPicture();
                    }
                    break;
                }
            }
        }

```

#### 拍摄图片

```java
private void captureStillPicture() {
        try {
            final Activity activity = getActivity();
            if (null == activity || null == mCameraDevice) {
                return;
            }
            // This is the CaptureRequest.Builder that we use to take a picture.
            final CaptureRequest.Builder captureBuilder =
                    mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_STILL_CAPTURE);
            captureBuilder.addTarget(mImageReader.getSurface());

            // Use the same AE and AF modes as the preview.
            captureBuilder.set(CaptureRequest.CONTROL_AF_MODE,
                    CaptureRequest.CONTROL_AF_MODE_CONTINUOUS_PICTURE);
            setAutoFlash(captureBuilder);

            // Orientation
            int rotation = activity.getWindowManager().getDefaultDisplay().getRotation();
            captureBuilder.set(CaptureRequest.JPEG_ORIENTATION, getOrientation(rotation));

            CameraCaptureSession.CaptureCallback CaptureCallback
                    = new CameraCaptureSession.CaptureCallback() {

                @Override
                public void onCaptureCompleted(@NonNull CameraCaptureSession session,
                                               @NonNull CaptureRequest request,
                                               @NonNull TotalCaptureResult result) {
                    showToast("Saved: " + mFile);
                    Log.d(TAG, mFile.toString());
                    unlockFocus();
                }
            };

            mCaptureSession.stopRepeating();
            mCaptureSession.abortCaptures();
            mCaptureSession.capture(captureBuilder.build(), CaptureCallback, null);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }
```

代码中有相应的注释。
注意Demo中的`onCaptureCompleted`完成后，又调用`unlockFocus()`解锁了焦点。是因为Demo做的是连续拍照，如果实现的是只拍一张照片，需要将`unlockFocus()`去掉。

#### 存图片

存图片的操作是在之前设置ImageReader的监听方法里面进行存储的。

```java
private final ImageReader.OnImageAvailableListener mOnImageAvailableListener
            = new ImageReader.OnImageAvailableListener() {

        @Override
        public void onImageAvailable(ImageReader reader) {
            mBackgroundHandler.post(new ImageSaver(reader.acquireNextImage(), mFile));
        }

    };
```

# 参考资料


[Android Camera2 使用总结](https://www.jianshu.com/p/73fed068a795)

[Android Camera2 简介](https://www.jianshu.com/p/23e8789fbc10)








