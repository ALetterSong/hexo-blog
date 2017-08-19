title: Android动画基础及点赞动画的实现
date: 2015-09-30 10:08:15
tags: "android"
---
   Android动画可分为**传统View动画(View Animation)**和**属性动画(Property Animation)**而View动画则分为**补间动画(Tween Animation)**和**帧动画(Frame Animation)**

**属性动画**，这个是在Android 3.0中才引进的，属性动画，它更改的是对象的实际属性，在View Animation（Tween Animation）中，其改变的是View的绘制效果，真正的View的属性保持不变，比如无论你在对话中如何缩放Button的大小，Button的有效点击区域还是没有应用动画时的区域，其位置与大小都不变。而在Property Animation中，改变的是对象的实际属性，如Button的缩放，Button的位置与大小属性值都改变了。而且Property Animation不止可以应用于View，还可以应用于任何对象。Property Animation只是表示一个值在一段时间内的改变，当值改变时要做什么事情完全是你自己决定的。
在Property Animation中，可以对动画应用以下属性：

 - Duration：动画的持续时间
 - TimeInterpolation：属性值的计算方式，如先快后慢
 - TypeEvaluator：根据属性的开始、结束值与TimeInterpolation计算出的因子计算出当前时间的属性值
 - Repeat Count and behavoir：重复次数与方式，如播放3次、5次、无限循环，可以此动画一直重复，或播放完时再反向播放
 - Animation sets：动画集合，即可以同时对一个对象应用几个动画，这些动画可以同时播放也可以对不同动画设置不同开始偏移
 - Frame refreash delay：多少时间刷新一次，即每隔多少时间计算一次属性值，默认为10ms，最终刷新时间还受系统进程调度与硬件的影响

 ``AnimatorSet``
提供组合动画能力的类。并可设置组中动画的时序关系，如同时播放、有序播放或延迟播放。Elevator会告诉属性动画系统如何计算一个属性的值，它们会从Animator类中获取时序数据，比如开始和结束值，并依据这些数据计算动画的属性值。
``ObjectAnimator``
继承自ValueAnimator，允许你指定要进行动画的对象以及该对象的一个属性。该类会根据计算得到的新值自动更新属性。也就是说上 Property Animation 的两个步骤都实现了。大多数的情况，你使用ObjectAnimator就足够了，因为它使得目标对象动画值的处理过程变得简单，不用再向ValueAnimator那样自己写动画更新的逻辑。但ObjectAnimator有一定的限制，比如它需要目标对象的属性提供指定的处理方法，这个时候你需要根据自己的需求在ObjectAnimator和ValueAnimator中做个选择了，看哪种实现更简便。
``插值器``
用于修改一个动画过程中的速率，可以定义各种各样的非线性变化函数，比如加速、减速等
在 Android 中所有的插值器都是 Interpolator 的子类，通过 android:interpolator 属性你可以引用不同的插值器。
``` 
    private void updateHeartButton(final int resId) {
        AnimatorSet animatorSet = new AnimatorSet();
        //旋转
        ObjectAnimator rotationAnim = ObjectAnimator.ofFloat(upDTO.getUpIv(), "rotation", 0f, 360f);
        rotationAnim.setDuration(300);
        //加速插值器
        rotationAnim.setInterpolator(ACCELERATE_INTERPOLATOR);
        //反弹
        ObjectAnimator bounceAnimX = ObjectAnimator.ofFloat(upDTO.getUpIv(), "scaleX", 0.2f, 1f);
        bounceAnimX.setDuration(300);
        //超出终点后的张力、拉力
        bounceAnimX.setInterpolator(OVERSHOOT_INTERPOLATOR);
        ObjectAnimator bounceAnimY = ObjectAnimator.ofFloat(upDTO.getUpIv(), "scaleY", 0.2f, 1f);
        bounceAnimY.setDuration(300);
        bounceAnimY.setInterpolator(OVERSHOOT_INTERPOLATOR);
        bounceAnimY.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationStart(Animator animation) {
                upDTO.getUpIv().setImageResource(resId);
            }
        });
 animatorSet.play(bounceAnimX).with(bounceAnimY).after(rotationAnim);
        animatorSet.start();
    }
```

