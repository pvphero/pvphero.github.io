---
title: 使用ZXing实现扫描多个条形码页面
date: 2019-01-03 17:08:54
tags:
- Android
categories:
- Android开发笔记
---

# 1.前言

ZXing是google官方推出的跨平台的基于Java实现处理扫面二维码或者条形码的库。支持很多格式，一维条码支持UPC-A，UPC-E，EAN-8，Code 39，Code 93等格式，二维条码支持QR Code，Data Matrix，PDF 417，MaxiCode等格式。

我们通常使用ZXing扫描的时候都是一个一个的去扫，但是用户的实际操作环境却不尽然。比如说下图：

![](https://ws3.sinaimg.cn/large/006tNc79ly1fyu9xmvmjgj30go08xn56.jpg)

<!--more-->

# 2.解决方案

ZXing中有一个类`GenericMultipleBarcodeReader`

看下这个类的源码：

```java
public final class GenericMultipleBarcodeReader implements MultipleBarcodeReader {

  private static final int MIN_DIMENSION_TO_RECUR = 100;
  private static final int MAX_DEPTH = 4;

  private final Reader delegate;

  public GenericMultipleBarcodeReader(Reader delegate) {
    this.delegate = delegate;
  }

  @Override
  public Result[] decodeMultiple(BinaryBitmap image) throws NotFoundException {
    return decodeMultiple(image, null);
  }

  @Override
  public Result[] decodeMultiple(BinaryBitmap image, Map<DecodeHintType,?> hints)
      throws NotFoundException {
    List<Result> results = new ArrayList<>();
    doDecodeMultiple(image, hints, results, 0, 0, 0);
    if (results.isEmpty()) {
      throw NotFoundException.getNotFoundInstance();
    }
    return results.toArray(new Result[results.size()]);
  }

  private void doDecodeMultiple(BinaryBitmap image,
                                Map<DecodeHintType,?> hints,
                                List<Result> results,
                                int xOffset,
                                int yOffset,
                                int currentDepth) {
    if (currentDepth > MAX_DEPTH) {
      return;
    }

    Result result;
    try {
      result = delegate.decode(image, hints);
    } catch (ReaderException ignored) {
      return;
    }
    boolean alreadyFound = false;
    for (Result existingResult : results) {
      if (existingResult.getText().equals(result.getText())) {
        alreadyFound = true;
        break;
      }
    }
    if (!alreadyFound) {
      results.add(translateResultPoints(result, xOffset, yOffset));
    }
    ResultPoint[] resultPoints = result.getResultPoints();
    if (resultPoints == null || resultPoints.length == 0) {
      return;
    }
    int width = image.getWidth();
    int height = image.getHeight();
    float minX = width;
    float minY = height;
    float maxX = 0.0f;
    float maxY = 0.0f;
    for (ResultPoint point : resultPoints) {
      if (point == null) {
        continue;
      }
      float x = point.getX();
      float y = point.getY();
      if (x < minX) {
        minX = x;
      }
      if (y < minY) {
        minY = y;
      }
      if (x > maxX) {
        maxX = x;
      }
      if (y > maxY) {
        maxY = y;
      }
    }

    // Decode left of barcode
    if (minX > MIN_DIMENSION_TO_RECUR) {
      doDecodeMultiple(image.crop(0, 0, (int) minX, height),
                       hints, results,
                       xOffset, yOffset,
                       currentDepth + 1);
    }
    // Decode above barcode
    if (minY > MIN_DIMENSION_TO_RECUR) {
      doDecodeMultiple(image.crop(0, 0, width, (int) minY),
                       hints, results,
                       xOffset, yOffset,
                       currentDepth + 1);
    }
    // Decode right of barcode
    if (maxX < width - MIN_DIMENSION_TO_RECUR) {
      doDecodeMultiple(image.crop((int) maxX, 0, width - (int) maxX, height),
                       hints, results,
                       xOffset + (int) maxX, yOffset,
                       currentDepth + 1);
    }
    // Decode below barcode
    if (maxY < height - MIN_DIMENSION_TO_RECUR) {
      doDecodeMultiple(image.crop(0, (int) maxY, width, height - (int) maxY),
                       hints, results,
                       xOffset, yOffset + (int) maxY,
                       currentDepth + 1);
    }
  }

  private static Result translateResultPoints(Result result, int xOffset, int yOffset) {
    ResultPoint[] oldResultPoints = result.getResultPoints();
    if (oldResultPoints == null) {
      return result;
    }
    ResultPoint[] newResultPoints = new ResultPoint[oldResultPoints.length];
    for (int i = 0; i < oldResultPoints.length; i++) {
      ResultPoint oldPoint = oldResultPoints[i];
      if (oldPoint != null) {
        newResultPoints[i] = new ResultPoint(oldPoint.getX() + xOffset, oldPoint.getY() + yOffset);
      }
    }
    Result newResult = new Result(result.getText(),
                                  result.getRawBytes(),
                                  result.getNumBits(),
                                  newResultPoints,
                                  result.getBarcodeFormat(),
                                  result.getTimestamp());
    newResult.putAllMetadata(result.getResultMetadata());
    return newResult;
  }

}
```

![](https://ws4.sinaimg.cn/large/006tNc79ly1fyu9y8z0lrj30kx0jpjuy.jpg)

可以看到`GenericMultipleBarcodeReader`中`decodeMultiple`返回 `Result[]`,所以我们可以根据这个类实现条形码的多个返回。

# 3.实现过程

## 3.1 gradle引入ZXing包

```java
implementation('com.journeyapps:zxing-android-embedded:3.6.0') { transitive = false }
    implementation 'com.google.zxing:core:3.3.2'
```

## 3.2 调用ZXing的扫描页面

```java
IntentIntegrator intentIntegrator = new IntentIntegrator(MainActivity.this);
                intentIntegrator
                        .setPrompt("")
                        .setBeepEnabled(false)
                        .setBarcodeImageEnabled(true)
                        .setDesiredBarcodeFormats(IntentIntegrator.CODE_128)
                        .initiateScan();
```

这里将`setBarcodeImageEnabled(true)`设置成true，就可以在activity的回调里面获取到扫描的二维码的路径。

## 3.3 在`onActivityResult`中处理

```java
@Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        IntentResult result = IntentIntegrator.parseActivityResult(requestCode, resultCode, data);
        LogUtils.i(TAG,result.toString());


        Bitmap bMap = BitmapFactory.decodeFile(result.getBarcodeImagePath());

        int[] data2 = new int[bMap.getWidth() * bMap.getHeight()];
        bMap.getPixels(data2, 0, bMap.getWidth(), 0, 0, bMap.getWidth(), bMap.getHeight());
        RGBLuminanceSource rgbLuminanceSource = new RGBLuminanceSource(bMap.getWidth(),bMap.getHeight(),data2);

        LuminanceSource source = rgbLuminanceSource;
        BinaryBitmap bitmap = new BinaryBitmap(new HybridBinarizer(source));
        Hashtable<DecodeHintType, Object> hints = new Hashtable<DecodeHintType, Object>    ();
        hints.put(DecodeHintType.TRY_HARDER, Boolean.TRUE);
        MultiFormatReader mreader = new MultiFormatReader();
        GenericMultipleBarcodeReader multireader = new GenericMultipleBarcodeReader(mreader);
        try {
            Result[] result2 = multireader.decodeMultiple(bitmap,hints);
            if(result != null){
                for(Result kp : result2)
                {
                    System.out.println(kp.toString());
                    LogUtils.i(TAG,kp.toString());
                }
            }

        } catch (NotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
```

# 4.运行效果

在log中我们可以看到扫描的结果
![](https://ws3.sinaimg.cn/large/006tNc79ly1fyu9xmvmjgj30go08xn56.jpg)

![](https://ws3.sinaimg.cn/large/006tNc79ly1fyu9z0ls83j30sy0he420.jpg)

# 5.结论

我们可以通过`GenericMultipleBarcodeReader`来实现扫面多个条形码，但是效果可能不是很稳定，这个识别的效果跟相机扫描出来的图片有关，先记下来，以后再好好优化下。






