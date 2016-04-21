layout: posts
title: Androidy启动页广告实现
date: 2016-04-21 16:50:10
tags:
    - Android
    - 移动开发
---
## 需求
启动页展示广告。主要功能：1.在启动的时候显示广告;2.【跳过】功能，倒计时3秒;3.查看广告详情。
如图所示：
![启动页广告](../../../../images/ad_1.png)

## 实现
layout布局文件：
``` xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- 广告图片 -->
    <ImageView
        android:id="@+id/splash_logo"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:visibility="visible"
        android:scaleType="fitXY"/>

    <!-- 跳过 -->
    <LinearLayout
        android:id="@+id/lin_skip"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:orientation="horizontal"
        android:layout_marginTop="40dp"
        android:padding="4dp"
        android:layout_marginRight="8dp"
        android:background="@drawable/splash_skip"
        android:visibility="gone">
        <TextView
            android:id="@+id/textView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:paddingLeft="5dp"
            android:layout_gravity="right"
            android:text="3"
            android:textColor="@color/white"
            android:textSize="16sp" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="right"
            android:paddingLeft="5dp"
            android:paddingRight="5dp"
            android:text="跳过"
            android:textColor="@color/white"
            android:textSize="16sp" />
    </LinearLayout>

</FrameLayout>
```

【跳过】按钮背景：
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <!-- 填充色 -->
    <solid android:color="#CBCBCB"/>
    <!-- 圆角 -->
    <corners android:radius="18dp"/>
</shape>
```

Activity文件:
``` java 
public class SplashActivity extends BaseActivity implements ISplashActivity {
    
    @Bind(R.id.splash_logo)
    ImageView splashLogo;
    @Bind(R.id.logo_layout)
    LinearLayout logoLayout;
    private Intent intent;
    @Bind(R.id.textView)
    TextView textView;
    @Bind(R.id.lin_skip)
    LinearLayout linSkip;
    private int count = 3;


    private ISplashPresenter splashPresenter;
    private HttpParams httpParams;
    private ADBean adBean;

    /**
     * 设置root界面
     */
    @Override
    public void setRootView() {
        super.setRootView();
        setContentView(R.layout.activity_splansh);
        ButterKnife.bind(this);
    }

    @Override
    public void initData() {
        super.initData();
    }

    @Override
    public void initWidget() {
        super.initWidget();
        init();
    }

    private void init() {
        logoLayout.setVisibility(View.VISIBLE);
        splashPresenter = new SplashPresenter(this);
        httpParams = new HttpParams();
        splashPresenter.getSplashAD(httpParams);
    }

    @Override
    public void onGetSplashADSuccess(ADBean adBean) {
        this.adBean = adBean;
        if (null != adBean && null != adBean.getData() && !"".equals(adBean.getData().getImagePath()) && null != adBean.getData().getImagePath()) {
            ImageLoader imageLoader = ImageLoader.getInstance();
            imageLoader.displayImage(API.FTP_SERVER + adBean.getData().getImagePath(), splashLogo, TrueLoveApp.options);
            linSkip.setVisibility(View.VISIBLE);
            handler.sendEmptyMessageDelayed(0, 1000);
        } else {
            splashLogo.setImageResource(R.mipmap.ic_splash);
            splashLogo.setClickable(false);
            linSkip.setVisibility(View.GONE);
            handler.sendEmptyMessageDelayed(-1, 1000);

        }

    }

    @OnClick(R.id.splash_logo)
    public void onSplashLogoClick(View view) {
        handler.removeMessages(0);
        startActivtiy();
    }

    @OnClick(R.id.lin_skip)
    public void OnSkipClick(View view) {
        handler.removeMessages(0);
        handler.sendEmptyMessageDelayed(1, 100);
    }

    @Override
    public void onGetSplashADError(String msg) {
        splashLogo.setImageResource(R.mipmap.ic_splash);
        splashLogo.postDelayed(new Runnable() {
            @Override
            public void run() {
                intent = new Intent(SplashActivity.this, MainActivity.class);
                startActivity(intent);
                finish();
            }
        }, CommonContant.SPLANSH_TIME);
    }

    private int getCount() {
        count--;
        if (count == 0) {
            Intent intent = new Intent(this, MainActivity.class);
            startActivity(intent);
            finish();
        }
        return count;
    }

    private Handler handler = new Handler() {
        public void handleMessage(android.os.Message msg) {
            if (msg.what == 0) {
                textView.setText(getCount()+"");
                handler.sendEmptyMessageDelayed(0, 1000);
            }
            // 跳过
            if (msg.what == 1) {
                handler.removeMessages(1);
                Intent intent = new Intent(SplashActivity.this, MainActivity.class);
                startActivity(intent);
                finish();
            }

            // 无广告
            if (msg.what == -1) {
                Intent intent = new Intent(SplashActivity.this, MainActivity.class);
                startActivity(intent);
                finish();
            }
        };
    };

    /**
     * Activity跳转
     */
    private void startActivtiy() {
        if (null != adBean) {
            if ("1".equals(adBean.getData().getRelatedContext())) { //shop
                Intent intent = new Intent(mContext, SellerShopDetailActivity.class);
                intent.putExtra("shopid", adBean.getData().getRelatedContentId());
                mContext.startActivity(intent);
            } else if ("2".equals(adBean.getData().getRelatedContext())) { //product
                Intent intent = new Intent(mContext, SellerProductDetailActivity.class);
                intent.putExtra("productid", adBean.getData().getRelatedContentId());
                mContext.startActivity(intent);
            } else {
                adBean.getData().setAdContent(new String(Base64.decode(adBean.getData().getAdContent(), Base64.DEFAULT)));
                Intent intent = new Intent(mContext, HomeADDetailActivity.class);
                intent.putExtra("flag", "splash");
                intent.putExtra("bean", adBean);
                mContext.startActivity(intent);
            }
        }
    }
}
```

## 运行效果
启动页：
![广告](../../../../images/ad.png)
广告详情：
![广告详情](../../../../images/ad_detail.png)