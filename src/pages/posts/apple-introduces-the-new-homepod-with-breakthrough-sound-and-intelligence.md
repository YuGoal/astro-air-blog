---
layout: '../../layouts/MarkdownPost.astro'
title: '全面屏虚拟导航栏适配（全面屏手势）'
pubDate: 2023-07-25
description: '为了实现在全面屏手势模式下，有小白条和没有小白条时底部显示不被遮盖'
author: 'caoyu'
cover:
    url: 'https://www.apple.com.cn/newsroom/images/product/homepod/standard/Apple-HomePod-hero-230118_big.jpg.large_2x.jpg'
    square: 'https://www.apple.com.cn/newsroom/images/product/homepod/standard/Apple-HomePod-hero-230118_big.jpg.large_2x.jpg'
    alt: 'cover'
tags: ["全面屏", "Android", "UI适配"] 
theme: 'light'
featured: true

meta:
 - name: caoyu
   content: 作者是我
 - name: keywords
   content: key3, key4

keywords: key1, key2, key3
---

## 适配工具类

```kotlin
import android.app.Service
import android.content.Context
import android.os.Build
import android.provider.Settings
import android.util.DisplayMetrics
import android.view.WindowManager
import com.lzf.easyfloat.utils.DisplayUtils

object NavigationUtils {

    /**
     * 获取虚拟导航栏(NavigationBar)的高度，可能未显示
     */
    fun getNavigationBarHeight(context: Context): Int {
        var result = 0
        val resources = context.resources
        val resourceId = resources.getIdentifier("navigation_bar_height", "dimen", "android")
        if (resourceId > 0) result = resources.getDimensionPixelSize(resourceId)
        return result
    }

    /**
     * 获取虚拟导航栏(NavigationBar)是否显示
     * @return true 表示虚拟导航栏显示，false 表示虚拟导航栏未显示
     */
    fun hasNavigationBar(context: Context) = when {
        getNavigationBarHeight(context) == 0 -> false
        RomUtils.checkIsHuaweiRom() && isHuaWeiHideNav(context) -> false
        RomUtils.checkIsMiuiRom() && isMiuiFullScreen(context) -> false
        RomUtils.checkIsVivoRom() && isVivoFullScreen(context) -> false
        else -> isHasNavigationBar(context)
    }

    /**
     * 华为手机是否隐藏了虚拟导航栏
     * @return true 表示隐藏了，false 表示未隐藏
     */
    private fun isHuaWeiHideNav(context: Context) =
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) {
            Settings.System.getInt(context.contentResolver, "navigationbar_is_min", 0)
        } else {
            Settings.Global.getInt(context.contentResolver, "navigationbar_is_min", 0)
        } != 0

    /**
     * 小米手机是否开启手势操作
     * @return true 表示使用的是手势，false 表示使用的是虚拟导航栏(NavigationBar)，默认是false
     */
    private fun isMiuiFullScreen(context: Context) =
        Settings.Global.getInt(context.contentResolver, "force_fsg_nav_bar", 0) != 0

    /**
     * Vivo手机是否开启手势操作
     * @return true 表示使用的是手势，false 表示使用的是虚拟导航栏(NavigationBar)，默认是false
     */
    private fun isVivoFullScreen(context: Context) =
        Settings.Secure.getInt(context.contentResolver, "navigation_gesture_on", 0) != 0

    /**
     * oppo手机是否开启手势操作
     * @return true 表示使用的是手势，false 表示使用的是虚拟导航栏(NavigationBar)，默认是false
     */
    private fun isOppoGestureEnabled(context: Context) =
        Settings.Secure.getInt(context.contentResolver, "navigation_gesture_on", 0) != 0



    /**
     * 根据屏幕真实高度与显示高度，判断虚拟导航栏是否显示
     * @return true 表示虚拟导航栏显示，false 表示虚拟导航栏未显示
     */
    private fun isHasNavigationBar(context: Context): Boolean {
        val windowManager: WindowManager =
            context.getSystemService(Service.WINDOW_SERVICE) as WindowManager
        val display = windowManager.defaultDisplay

        val realDisplayMetrics = DisplayMetrics()
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            display.getRealMetrics(realDisplayMetrics)
        }
        val realHeight = realDisplayMetrics.heightPixels
        val realWidth = realDisplayMetrics.widthPixels

        val displayMetrics = DisplayMetrics()
        display.getMetrics(displayMetrics)
        val displayHeight = displayMetrics.heightPixels
        val displayWidth = displayMetrics.widthPixels

        // 部分无良厂商的手势操作，显示高度 + 导航栏高度，竟然大于物理高度，对于这种情况，直接默认未启用导航栏
        if (displayHeight > displayWidth) {
            if (displayHeight + DisplayUtils.getNavigationBarHeight(context) > realHeight) return false
        } else {
            if (displayWidth + DisplayUtils.getNavigationBarHeight(context) > realWidth) return false
        }

        return realWidth - displayWidth > 0 || realHeight - displayHeight > 0
    }
}
```

## 工具类RomUtils

```kotlin
object RomUtils {
    private const val TAG = "RomUtils--->"

    /**
     * 获取 emui 版本号
     */
    @JvmStatic
    fun getEmuiVersion(): Double {
        try {
            val emuiVersion = getSystemProperty("ro.build.version.emui")
            val version = emuiVersion!!.substring(emuiVersion.indexOf("_") + 1)
            return version.toDouble()
        } catch (e: Exception) {
            e.printStackTrace()
        }
        return 4.0
    }

    @JvmStatic
    fun getSystemProperty(propName: String): String? {
        val line: String
        var input: BufferedReader? = null
        try {
            val p = Runtime.getRuntime().exec("getprop $propName")
            input = BufferedReader(InputStreamReader(p.inputStream), 1024)
            line = input.readLine()
            input.close()
        } catch (ex: Exception) {
            Log.e(TAG, "Unable to read sysprop $propName", ex)
            return null
        } finally {
            if (input != null) {
                try {
                    input.close()
                } catch (e: IOException) {
                    Log.e(TAG, "Exception while closing InputStream", e)
                }
            }
        }
        return line
    }

    fun checkIsHuaweiRom() = Build.MANUFACTURER.contains("HUAWEI")

    fun checkIsMiuiRom() = !TextUtils.isEmpty(getSystemProperty("ro.miui.ui.version.name"))

    fun checkIsMeizuRom(): Boolean {
        val systemProperty = getSystemProperty("ro.build.display.id")
        return if (TextUtils.isEmpty(systemProperty)) false
        else systemProperty!!.contains("flyme") || systemProperty.toLowerCase().contains("flyme")
    }

    fun checkIs360Rom(): Boolean =
        Build.MANUFACTURER.contains("QiKU") || Build.MANUFACTURER.contains("360")

    fun checkIsOppoRom() =
        Build.MANUFACTURER.contains("OPPO") || Build.MANUFACTURER.contains("oppo")

    fun checkIsVivoRom() =
        Build.MANUFACTURER.contains("VIVO") || Build.MANUFACTURER.contains("vivo")

}
```

## 使用场景

```kotlin
if (NavigationUtils.hasNavigationBar(this@JsBridgeWebViewActivity)){
                val layoutParams = mViewBinding.jsBridgeWebView.layoutParams as ViewGroup.MarginLayoutParams
                layoutParams.bottomMargin = BarUtils.getNavBarHeight()
                mViewBinding.jsBridgeWebView.layoutParams = layoutParams
}
```

BarUtils.getNavBarHeight()或取手机底部导航栏高度。
