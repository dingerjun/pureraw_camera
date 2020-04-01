package com.mobile.activity;

import android.app.Activity;
import android.annotation.SuppressLint;
import android.app.Activity;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Matrix;
import android.graphics.Paint;
import android.graphics.PixelFormat;
import android.graphics.PorterDuffXfermode;
import android.graphics.Paint.Style;
import android.graphics.PorterDuff.Mode;
import android.hardware.Camera;
import android.hardware.Camera.PreviewCallback;
import android.util.DisplayMetrics;
import android.os.Bundle;
import android.os.Handler;
import android.os.IBinder;
import android.os.Message;
import android.os.RemoteException;
import android.os.SystemClock;
import android.util.Log;
import android.view.Surface;
import android.view.SurfaceHolder;
import android.view.SurfaceView;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.Window;
import android.view.WindowManager;
import android.widget.Button;
import android.widget.Toast;

import com.mobilechoose.R;

import java.util.List;
import java.io.FileOutputStream;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.IOException;

import android.os.SystemProperties;

@SuppressLint("WrongCall")
public class IrisAppUnlockActiviy extends Activity implements
SurfaceHolder.Callback, PreviewCallback {
    private static final String TAG = "IrisDemoActiviy";
    private SurfaceView mySurfaceView = null;
    private SurfaceHolder mySurfaceHolder = null;
    private SurfaceView mySurfaceView2 = null;
    private SurfaceHolder mySurfaceHolder2 = null;
    private Camera myCamera = null;
    private Camera.Parameters myParameters;
    private boolean isView = false;
    static private Bitmap mBitmap;
    private int mSensorWidth =0;
    private int mSensorHeight =0;
    int[] pixels = new int[1920*1080/4]; // buffer for unpacked bmp

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        int flag = WindowManager.LayoutParams.FLAG_FULLSCREEN;
        // android:screenOrientation="portrait" //ver
        // android:screenOrientation="landscape"//hes
        Window myWindow = this.getWindow();
        myWindow.setFlags(flag, flag);
        
        myWindow.setFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON,
                          WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
        DisplayMetrics dm = new DisplayMetrics();
        getWindowManager().getDefaultDisplay().getMetrics(dm);
        Log.d(TAG,String.format("pix=" + dm.widthPixels + "X" + dm.heightPixels));
        setContentView(R.layout.main);
        initView();
    }

    private void initView() {
        mySurfaceView = (SurfaceView) findViewById(R.id.mySurfaceView);
        mySurfaceHolder = mySurfaceView.getHolder();
        mySurfaceHolder.setFormat(PixelFormat.TRANSLUCENT);
        mySurfaceHolder.addCallback(this);
        mySurfaceHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
        
        mySurfaceView2 = (SurfaceView) findViewById(R.id.mySurfaceView2);
        mySurfaceHolder2 = mySurfaceView2.getHolder();
        mySurfaceView2.setZOrderOnTop(true);
        mySurfaceView2.setVisibility(View.VISIBLE);
    }

    // Draw the unpacked photo
    private void SimpleDraw() {
        if (mBitmap == null) return;
        Canvas c = null;
        c = mySurfaceHolder2.lockCanvas(null);
        c.drawColor(Color.BLACK);
        c.drawBitmap(mBitmap, 0, 0, null);

        mySurfaceHolder2.unlockCanvasAndPost(c);
    }

    protected void onResume() {
        super.onResume();
        mSensorWidth = SystemProperties.getInt("iris.sensorwidth", 1080);
        mSensorHeight = SystemProperties.getInt("iris.sensorheight", 1920);
        mBitmap = Bitmap.createBitmap(mSensorHeight/2, mSensorWidth/2, Bitmap.Config.ARGB_8888);
        initCamera();
        // Log.e(TAG, String.format("loader irisjni\n"));
    }

    public void surfaceChanged(SurfaceHolder arg0, int arg1, int arg2, int arg3) {
        Log.d(TAG, "surfaceChanged");
        mySurfaceHolder = arg0;
        if (myCamera != null) {
            try {
                myCamera.setPreviewDisplay(mySurfaceHolder);
                myCamera.setPreviewCallback(this);
                myCamera.startPreview();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public void surfaceCreated(SurfaceHolder arg0) {
        Log.d(TAG, "surfaceCreated");
    }

    public void surfaceDestroyed(SurfaceHolder arg0) {
        Log.d(TAG, "surfaceDestroyed");
        if (myCamera != null) {
            myCamera.stopPreview();
        }
        mySurfaceHolder = null;
    }

    // return the id of front camera
    private int FindFrontCamera() {
        int cameraCount = 0;
        Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
        cameraCount = Camera.getNumberOfCameras(); // get cameras number
        
        for (int camIdx = 0; camIdx < cameraCount; camIdx++) {
            Camera.getCameraInfo(camIdx, cameraInfo); // get camerainfo
            if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
                return camIdx;
            }
        }
        return -1;
    }

    // return the id of back camera
    private int FindBackCamera() {
        int cameraCount = 0;
        Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
        cameraCount = Camera.getNumberOfCameras(); // get cameras number
        
        for (int camIdx = 0; camIdx < cameraCount; camIdx++) {
            Camera.getCameraInfo(camIdx, cameraInfo); // get camerainfo
            if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_BACK) {
                return camIdx;
            }
        }
        return -1;
    }

    private static int getDisplayRotation(Activity activity) {
        int rotation = activity.getWindowManager().getDefaultDisplay()
        .getRotation();
        switch (rotation) {
            case Surface.ROTATION_0:
                return 0;
            case Surface.ROTATION_90:
                return 90;
            case Surface.ROTATION_180:
                return 180;
            case Surface.ROTATION_270:
                return 270;
        }
        return 0;
    }

    private static void setCameraDisplayOrientation(Activity activity,
                                                    int cameraId, Camera camera) {
        // See android.hardware.Camera.setCameraDisplayOrientation for
        // documentation.
        Camera.CameraInfo info = new Camera.CameraInfo();
        Camera.getCameraInfo(cameraId, info);
        int degrees = getDisplayRotation(activity);
        int result;
        if (info.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
            result = (info.orientation + degrees) % 360;
            result = (360 - result) % 360; // compensate the mirror
        } else { // back-facing
            result = (info.orientation - degrees + 360) % 360;
        }
        camera.setDisplayOrientation(result);
    }
    
    // private final class YuvPreviewCallback implements
    // android.hardware.Camera.PreviewCallback
    // This is the preview callback with YUV420 format.
    // The buffer size of data should be 1920x1080 x (1 + 1/4 + 1/4) = 3,110,400
    // The buffer is overwrite with 10bit packed raw, valid size should be: 1920x1080x10/8 = 2,592,000
    // The packed format should be:
    //  ------------------------------------------------------------------------------------------------------------------------
    // |            Byte 0      |          Byte 1       |          Byte 2       |          Byte 3       |          Byte 4       |
    //  ------------------------------------------------------------------------------------------------------------------------
    // | b0 b1 b2 b3 b4 b5 b6 b7|b0 b1 b2 b3 b4 b5 b6 b7|b0 b1 b2 b3 b4 b5 b6 b7|b0 b1 b2 b3 b4 b5 b6 b7|b0 b1 b2 b3 b4 b5 b6 b7|
    //  ------------------------------------------------------------------------------------------------------------------------
    // | p0 p1 p2 p3 p4 p5 p6 p7 p8 p9|p0 p1 p2 p3 p4 p5 p6 p7 p8 p9|p0 p1 p2 p3 p4 p5 p6 p7 p8 p9|p0 p1 p2 p3 p4 p5 p6 p7 p8 p9|
    //  ------------------------------------------------------------------------------------------------------------------------
    // |            pixel 0           |              pixel 1           |              pixel 2     |              pixel 3        |
    //  ------------------------------------------------------------------------------------------------------------------------
    // NOTE: each line will be independent, the line size of packed for 1080 is 1352 bytes, while for 1920 is 2400
    // unpack example of a 4 pixels with 5 bytes in C code
    // byte0 = *(buf++);
    // byte1 = *(buf++);
    // byte2 = *(buf++);
    // byte3 = *(buf++);
    // byte4 = *(buf++);
    // bits10_pixel0 = (byte0 +((byte1 & 0x03) << 8));
    // bits10_pixel1 = (((byte1 & 0xFC) >> 2)+(((byte2 & 0xF) << 6)));
    // bits10_pixel2 = (((byte2 & 0xF0) >> 4)+((byte3 & 0x3F) << 4));
    // bits10_pixel3 = (((byte3 & 0xC0) >> 6)+(byte4 << 2));
    public void onPreviewFrame(byte[] data, Camera camera) {
        // TODO Auto-generated method stub
        Log.e(TAG, "onPreviewFrame");
        covertToBitmap(data);
        SimpleDraw();
    }

    // Save unpacked display image for debug
    private void saveBitmap(Bitmap bitmap) {
        Log.e(TAG, "save unpacked frame");
        File f = new File("/sdcard/iris_frame.png");
        if (f.exists()) {
            f.delete();
        }
        try {
            FileOutputStream out = new FileOutputStream(f);
            bitmap.compress(Bitmap.CompressFormat.PNG, 90, out);
            out.flush();
            out.close();
            Log.i(TAG, "saved unpacked frame");
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

    private void saveRaw(byte[] buf) {
        Log.e(TAG, "save unpacked frame");
        File f = new File("/sdcard/iris_frame.raw");
        if (f.exists()) {
            f.delete();
        }
        try {
            FileOutputStream out = new FileOutputStream(f);
            out.write(buf, 0, 1920*1080*10/8);
            out.flush();
            out.close();
            Log.i(TAG, "saved unpacked frame");
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

    // convert buffer to bitmap: here the unpack is sub-sampling to 960x540 for fast display
    // Note: Clockwise 90 degree applied for unpack output, due to there is a -90 degree rotation of HW compared with display direction.
    private void covertToBitmap(byte[] yuvData) {
        if (mBitmap == null) return;
        Log.d(TAG, "w: " + mSensorWidth + "h: " + mSensorHeight);

        // stride for 1080 line width is 1352, while 2400 for 1920 line width
        int stride = 1352;
        //int stride = 2400;
        //subsampling and rotate 90 degree
        int dispH = mSensorWidth/2;
        int dispW = mSensorHeight/2;
        for (int i = 0; i < dispW; i++) {
            int head = (2 * i) * stride;
            for (int j = 0; j < mSensorWidth/4; j++) {
                int k = head + j * 5;// 5 bytes each 4pixel
                int byte0 = yuvData[k++] & 0xFF;
                int byte1 = yuvData[k++] & 0xFF;
                int byte2 = yuvData[k++] & 0xFF;
                int byte3 = yuvData[k++] & 0xFF;
                //int byte4 = yuvData[k++] & 0xFF; // subsample for display

                k = 2 * j * dispW + (dispW -1 - i);
                int grey = (byte0 +((byte1 & 0x3) << 8))>>2;
                pixels[k] = 0xFF000000 | (grey * 0x00010101);
                //pixels[k++] = 0xFF000000 | (grey * 0x00010101);

                //grey = (((byte1 & 0xFC) >> 2)+(((byte2 & 0xf) << 6)))>>2;
                //pixels[k++] = 0xFF000000 | (grey * 0x00010101);

                k += dispW;
                grey = (((byte2 & 0xf0) >> 4)+((byte3 & 0x3f) << 4))>>2;
                pixels[k] = 0xFF000000 | (grey * 0x00010101);
                //pixels[k++] = 0xFF000000 | (grey * 0x00010101);

                //grey = (((byte3 & 0xc0) >> 6)+(byte4 << 2))>>2;
                //pixels[k++] = 0xFF000000 | (grey * 0x00010101);
            }
        }

        mBitmap.setPixels(pixels, 0, dispW, 0, 0, dispW, dispH);
        //saveBitmap(bitmap);
        //saveRaw(yuvData);
    }

    private void initCamera() {
        int CameraIndex = -1;
        CameraIndex = FindFrontCamera();
        if (CameraIndex == -1) {
            CameraIndex = FindBackCamera();
            if (CameraIndex == -1) {
                Log.e(TAG, "no camera device");
            }
        }
        if (myCamera == null && !isView) {
            myCamera = Camera.open(CameraIndex);
            myCamera.setPreviewCallback(this);
            Log.i(TAG, "camera.open success\n");
        }
        if (myCamera != null && !isView) {
            try {
                myCamera.stopPreview();
                myParameters = myCamera.getParameters();

                List<Camera.Size> sizeList = myParameters
                .getSupportedPreviewSizes();
                
                for (int i = 1; i < sizeList.size(); i++) {
                    Log.i(TAG, "support " + sizeList.get(i).width + "x" + sizeList.get(i).height);
                }

                myParameters.setPreviewSize(1080, 1920);
                myParameters.setPreviewFpsRange(20000, 30000);
                myCamera.setParameters(myParameters);
                myCamera.setPreviewDisplay(mySurfaceHolder);
                myCamera.startPreview();
                isView = true;
                // Log.d(TAG, String.format("set Parameters success\n"));
            } catch (Exception e) {
                // TODO: handle exception
                e.printStackTrace();
                Toast.makeText(IrisAppUnlockActiviy.this, getResources().getString(R.string.Initialize_camera_error),Toast.LENGTH_SHORT).show();
            }
        }
    }

    protected void onPause() {
        Log.e(TAG, String.format("onPause: ready to pause app\n"));
        //myParameters = myCamera.getParameters();
        //myParameters.setFlashMode(Camera.Parameters.FLASH_MODE_OFF);
        //myCamera.setParameters(myParameters);

        if (myCamera != null) {
            myCamera.stopPreview();
            myCamera.setPreviewCallback(null);
            myCamera.release();
            myCamera = null;
        }
        mBitmap = null;
        mSensorWidth = 0;
        mSensorHeight = 0;
        isView = false;
        super.onPause();
    }
}

