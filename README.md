# puctureClipTest
一个获取手机本地相册或者拍照获得图片进行裁剪的示例代码
一.选择本地图片进行裁剪编辑

    1.使用意图Intent.action.PICK (媒体库列表，选择某项并返回所选数据)

    (intent.action.GET_CONTENT 打开的是文件系统（包括视频、音频、图片等）供用户选择数据，并返回所选数据)


Intent mIntent = new Intent(Intent.ACTION_PICK, MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
startActivityForResult(mIntent,ROAD_PICTURE);

第二个参数是为了筛选出媒体库里的图片资源文件，

2.通过onActivityResult()方法中返回的data获取选中图片的uri

调用下面方法进行图片的裁剪

private String startPhotoZoom(Uri uri) {
        try {
            Log.e("====", "uri = " + uri);

            SimpleDateFormat sDateFormat = new SimpleDateFormat(
                    "yyyyMMddhhmmss");
            String address = sDateFormat.format(new java.util.Date());
            if (!FileUtils.isFileExist("")) {
                FileUtils.createSDDir("");

            }
            imageUrl = FileUtils.SDPATH + address + ".JPEG";
            Log.e(TAG,"imageUrl ="+imageUrl);
            //1.从Uri获得文件路径
            Uri imageUri = Uri.fromFile(new File(imageUrl));

            //调用以下代码会跳转到Android系统自带的一个图片剪裁页面，点击确定之后就会得到一张图片
            final Intent intent = new Intent("com.android.camera.action.CROP");
            intent.setDataAndType(uri, "image/*");
            intent.putExtra("crop", "true"); //cropString 发送裁剪信号
            intent.putExtra("aspectX", 1);     //aspectXintX 方向上的比例
            intent.putExtra("aspectY", 1);  // aspectYintY 方向上的比例
            intent.putExtra("outputX", 480);//outputXint  裁剪区的宽
            intent.putExtra("outputY", 480); //outputYint 裁剪区的高
            intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
            intent.putExtra("outputFormat",
                    Bitmap.CompressFormat.JPEG.toString());
            intent.putExtra("noFaceDetection", false);
            intent.putExtra("return-data", false);
            startActivityForResult(intent, CUT_PHOTO_REQUEST_CODE);
            return address;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}

3.进入系统调用的裁剪界面点确定按键后，可以根据配置生成图片文件

即调用下面代码

if (resultCode == RESULT_OK && null != data) {
    Bitmap bitmap = ImageCacheUtil.getLoacalBitmap(imageUrl);
    System.out.println("bitmap = " + bitmap);
    FileUtils.deleteDir(FileUtils.SDPATH);
    uploadBitmap = ImageCacheUtil.createFramedPhoto(480, 480,
            bitmap, (int) (dp * 1.6f)); //480*480 是保存图片的分辨率
    FileUtils.saveBitmap(uploadBitmap, picName);
    File file = new File(FileUtils.SDPATH, picName + ".JPEG");

    //获取到这个file 文件后进行网络上传等操作，这里只是显示界面上！
    if(file ==null){
        Log.e(TAG,"file == null");}
    else if(file != null)
    {
        Log.e(TAG,"file != null");
        //show_cot_imv.setBackgroundResource(R.mipmap.ic_launcher);
        //ImageCacheUtil.toRoundBitmap（）；进行圆角化处理
 show_cot_imv.setImageBitmap(ImageCacheUtil.toRoundBitmap(uploadBitmap));
        Log.e(TAG,"filedir ="+FileUtils.SDPATH+picName+".JPEG");
        /*String fileName =FileUtils.SDPATH+picName+".JPEG" ;
        Bitmap bm = BitmapFactory.decodeFile(fileName);
        show_cot_imv.setImageBitmap(bm);*/
    }

}

4.最后得到保存在

FileUtils.SDPATH+picName+".JPEG"路劲下的裁剪图片。

二.通过拍照获取图片进行裁剪

1.使用 MediaStore.ACTION_IMAGE_CAPTURE 启动安装在手机上的摄像头应用程序

直接看代码

try {
    Intent openCameraIntent = new Intent(
            MediaStore.ACTION_IMAGE_CAPTURE);

    String sdcardState = Environment.getExternalStorageState();
    String sdcardPathDir = android.os.Environment
            .getExternalStorageDirectory().getPath() + "/tempImage/";
    File file = null;
    if (Environment.MEDIA_MOUNTED.equals(sdcardState)) {
        // ��sd�����Ƿ���myImage�ļ���
        File fileDir = new File(sdcardPathDir);
        if (!fileDir.exists()) {
            fileDir.mkdirs();
        }
        // �Ƿ���headImg�ļ�
        file = new File(sdcardPathDir + System.currentTimeMillis()
                + ".JPEG");
    }
    if (file != null) {
        path = file.getPath();
        photoUri = Uri.fromFile(file);
        Log.e("=====", "��ȡ����ͼƬ��ַ ��" + photoUri);
        openCameraIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoUri);

        startActivityForResult(openCameraIntent, TAKE_PICTURE);
    }

} catch (Exception e) {
    e.printStackTrace();
}

点确定后，onActivityResult()中把photoUri 传递给startPhotoZoom();
