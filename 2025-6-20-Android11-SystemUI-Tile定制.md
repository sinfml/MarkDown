---
layout:     post
title:      Android11-SystemUI定制
subtitle:   如何对Android11的SystemUI进行定制
date:       2025-6-20
author:     sinfml
catalog: true
tags:
    - Android 
    - FrameWork
    - SystemUI
---

## 前提

针对如何对Android11的SystemUI进行定制，增加自定义Tile。

## 增加截屏Tile

### 1.在`app\src\main\res\values`的config.xml中增加

```xml
    //添加screenshot代表截图控件的alias
    //快捷菜单前台默认选项
    <string name="quick_settings_tiles_defaut">
        location, screenshot
    </string>

    //快捷菜单后台默认选项
    <string name=“quick_settings_tiles_stock”>
        location, screenshot
    </string>
```

### 2.增加ScreenShotControllerImpl类

用于截图的主要操作逻辑

```java

    @Inject
    public ScreenShotControllerImpl(Context context, GlobalScreenshot globalScreenshot, @Main Handler mainHandler) {
        mContext = context;
        mGlobalScreenshot = globalScreenshot; // 截图工具
        mMainHandler = mainHandler; //住Handler
    }

    // 点击Tile按钮触发，在此实现截图逻辑
    public void setScreenShotController(boolean enabled) {
        synchronized(this) {
            // ...

            // 截图前，隐藏下拉栏
            Dependency.get(CommandQueu.class).animteCollapsePanels();

            // 等待500ms，待下拉栏隐藏后开始截图
            mMainHandler.postDelayed(() -> {
                mGlobalScreenShot.takeScreenshot(
                    // 截图保存后的uri连接
                    uri -> {
                        if (uri != null) {
                            // 通知媒体库更新
                            mContext.sendBroadcast(new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE, uri));
                        }
                    },
                    () -> {
                        // onCompleted
                    }
                );
            }, 500);
        }
    }

```

### 3.增加ScreenShotTile

该类用于下拉栏快捷图标按钮

```java
public class ScreenShotTile extends QSTileImpl<QSTile.BooleanState> {

    private final Icon startIcon = ...
    private final Icon stopIcon = ...

    private final ScreenShotContoller mController;

    @Inject
    protected ScreenShotTile(QSHost host, ScreenShotController screenShotController) {
        super(host);
        this.mController = screenShotController;
    }

    @Override
    public BooleanState newTileState() {
        return new BooleanState();
    }

    @Override
    protected void handleClick() {
        ...

        //处理点击事件
        boolean newState = !mState.value;
        //触发截屏操作
        mController.setScreenShotController(newState);
        refreshState(newState);
    }

    @Override
    protected void handleUpdateState(BooleanState state, Object arg) {
        mState = state;
        if (state.slash == null) {
            state.slash = new SlashState();
        }

        //标题
        state.label = ...
        if (!mController.isAvailable()) {
            state.icon = startIcon;
            state.slash.isSlashed = true;
            state.controlDescription = ...
            state.state = Tile.STATE_UNAVAILABLE;
            return;
        }

        if (state.value) {
            state.icon = stopIcon;
        } else {
            state.icon = startIcon;
        }

        ...
    }

    @Override
    public ChareSequence getTileLabel() {
        // 标题
        return ...
    }
}

```


### 4.在`QSFactoryImpl`增加对应的Provider

```java
    private final Provider<ScreenShotTile> mScreenShotTileProvider;

    @Inject
    public QSFactoryImpl(
        ...
        Provider<ScreenShotTile> screenShotTileProvider
    ) {
        ...
        mScreenShotTileProvider = screenShotTileProvider;
    }

    private QSTileImpl createTileInternal(String tileSpec) {
        switch (tileSpec) {
            case "location":
                return mLocationTileProvider.get();
                // 关联截屏的alias
            case "screenshot":
                return mScreenShotTileProvider.get();
        }
    }
```

### 5.在注入框架`DependencyBinder.java`除绑定`ScreenShotController`

```java
    @Binds
    public abstract ScreenShotController provideScreenShotController(ScreenShotControllerImpl controllerImpl);
```

### 6.调整截屏后弹出的截图预览框的分享以及调整按钮

隐藏分享与设置按钮

```java
    private ValueAnimator createScreenshotActionShadeAnimation(SavedImageData imageData) {
        LayoutInflater inflater = LayoutInflater.from(mContext);

        ...
             for (Notification.Action smartAction : imageData.smartActions) {
            ScreenshotActionChip actionChip = (ScreenshotActionChip) inflater.inflate(
                    R.layout.global_screenshot_action_chip, mActionsView, false);
            actionChip.setText(smartAction.title);
            actionChip.setIcon(smartAction.getIcon(), false);
            actionChip.setPendingIntent(smartAction.actionIntent,
                    () -> {
                        mUiEventLogger.log(ScreenshotEvent.SCREENSHOT_SMART_ACTION_TAPPED);
                        dismissScreenshot("chip tapped", false);
                        mOnCompleteRunnable.run();
                    });
            mActionsView.addView(actionChip);
            chips.add(actionChip);
        }

        // 注释，隐藏分享按钮以及调整按钮
        /*ScreenshotActionChip shareChip = (ScreenshotActionChip) inflater.inflate(
                R.layout.global_screenshot_action_chip, mActionsView, false);
        shareChip.setText(imageData.shareAction.title);
        shareChip.setIcon(imageData.shareAction.getIcon(), true);
        shareChip.setPendingIntent(imageData.shareAction.actionIntent, () -> {
            mUiEventLogger.log(ScreenshotEvent.SCREENSHOT_SHARE_TAPPED);
            dismissScreenshot("chip tapped", false);
            mOnCompleteRunnable.run();
        });
        mActionsView.addView(shareChip);
        chips.add(shareChip);

        ScreenshotActionChip editChip = (ScreenshotActionChip) inflater.inflate(
                R.layout.global_screenshot_action_chip, mActionsView, false);
        editChip.setText(imageData.editAction.title);
        editChip.setIcon(imageData.editAction.getIcon(), true);
        editChip.setPendingIntent(imageData.editAction.actionIntent, () -> {
            mUiEventLogger.log(ScreenshotEvent.SCREENSHOT_EDIT_TAPPED);
            dismissScreenshot("chip tapped", false);
            mOnCompleteRunnable.run();
        });
        mActionsView.addView(editChip);
        chips.add(editChip); */

        ...

        // remove the margin from the last chip so that it's correctly aligned with the end
        /*LinearLayout.LayoutParams params = (LinearLayout.LayoutParams)
                mActionsView.getChildAt(mActionsView.getChildCount() - 1).getLayoutParams();*/
        // 因隐藏了两个按钮， 这块会出现空指针异常
        View lastActionView = mActionsView.getChildCount() > 0 ? mActionsView.getChildAt(mActionsView.getChildCount() - 1) : null;
        if (lastActionView != null) {
            LinearLayout.LayoutParams params = lastActionView.getLayoutParams();
            params.setMarginEnd(0);
        }

        ...
        // 隐藏两个按钮的背景
        /*animator.addUpdateListener(animation -> {
            float t = animation.getAnimatedFraction();
            mBackgroundProtection.setAlpha(t);
            float containerAlpha = t < alphaFraction ? t / alphaFraction : 1;
            mActionsContainer.setAlpha(containerAlpha);
            mActionsContainerBackground.setAlpha(containerAlpha);
            float containerScale = SCREENSHOT_ACTIONS_START_SCALE_X
                    + (t * (1 - SCREENSHOT_ACTIONS_START_SCALE_X));
            mActionsContainer.setScaleX(containerScale);
            mActionsContainerBackground.setScaleX(containerScale);
            for (ScreenshotActionChip chip : chips) {
                chip.setAlpha(t);
                chip.setScaleX(1 / containerScale); // invert to keep size of children constant
            }
            mActionsContainer.setScrollX(mDirectionLTR ? 0 : mActionsContainer.getWidth());
            mActionsContainer.setPivotX(mDirectionLTR ? 0 : mActionsContainer.getWidth());
            mActionsContainerBackground.setPivotX(
                    mDirectionLTR ? 0 : mActionsContainerBackground.getWidth());
        });*/   
    }

```
