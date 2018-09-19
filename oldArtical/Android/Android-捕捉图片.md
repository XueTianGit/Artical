title: Android-捕捉图片
date: 7/24/2016 10:44:34 AM       
categories: Android
---

#调用系统相机获得高清图片
>代码：


    private static final int CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE = 100;

 	private void captureCamera() {
        //拍照
        Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);

        //指定original.jpg保存的位置
        labelPhotoPath = Constant.DATA_PHOTO_DIR + File.separator + taskName + File.separator + labelNumber;

        SDCardUtils.createDir(labelPhotoPath);
        originalImageUri = Uri.fromFile(new File(labelPhotoPath + File.separator + "original.jpg"));
        intent.putExtra(MediaStore.EXTRA_OUTPUT, originalImageUri);
        startActivityForResult(intent, CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE);
    }

	@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        // 如果是拍照
        if (CAPTURE_IMAGE_ACTIVITY_REQUEST_CODE == requestCode) {
            if (RESULT_OK == resultCode) {
      			//照片生成成功，并且已经保存到指定目录
            } else if (resultCode == RESULT_CANCELED) {
                // User cancelled the image capture
            }
        }
    }


#将高清图片压缩为质量较差的图片
>上面通过这种方法获得的图片质量太高(图片太大)， 如果我们使用不到这么大的图片，我们需要对图片进行压缩之后，再使用

>代码

	 public static void writeBitmapToPhotoWorkPath(Bitmap bitmap, String path, int pressPercent) {
        File originalFile = new File(path);
        SDCardUtils.writeBitMapToFile(bitmap, originalFile, pressPercent);  //压缩比例  0 - 100;  100为保持原图，默认不压缩
    }



	public static boolean writeBitMapToFile(Bitmap originalBitmap, File originalFile, int pressPercent) {
        boolean ret = false;
        try {
            OutputStream bitMapOutputStream = new FileOutputStream(originalFile);
            originalBitmap.compress(Bitmap.CompressFormat.JPEG, pressPercent, bitMapOutputStream);
            bitMapOutputStream.close();
            return ret;

        } catch (Exception e) {
            System.out.println("压缩文件出现异常");
        }

        return ret;
    }


#获取ImageView中的Bitmap对象
>代码

	 //获取ImageView中的Bitmap
    public Bitmap getImageViewBitMap(){
        this.setDrawingCacheEnabled(true);
        Bitmap tempBitmap = Bitmap.createBitmap(this.getDrawingCache());  //不能直接返回 ： this.getDrawingCache()
        this.setDrawingCacheEnabled(false);
        return tempBitmap;

    }