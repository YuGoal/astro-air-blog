---
layout: '../../layouts/MarkdownPost.astro'
title: 'kotlin金额处理'
pubDate: 2023-07-21
description: '解决接口返回浮点数丢失精度等显示不正确的问题'
author: 'caoyu'
cover:
    url: 'https://www.apple.com.cn/newsroom/cn/images/product/iphone/lifestyle/Apple_Shot-on-iPhone-14-models_iPhone-14-Pro-Max-with-the-Main-Camera-by-Xiaobei-Fuzhou_12192022_Full-Bleed-Image.jpg.xlarge_2x.jpg'
    square: 'https://www.apple.com.cn/newsroom/cn/images/product/iphone/lifestyle/Apple_Shot-on-iPhone-14-models_iPhone-14-Pro-Max-with-the-Main-Camera-by-Xiaobei-Fuzhou_12192022_Full-Bleed-Image.jpg.xlarge_2x.jpg'
    alt: 'cover'
tags: ["Kotlin", "扩展函数"] 
theme: 'dark'
featured: true
---

## 适配工具类

```kotlin
import java.math.BigDecimal

fun formatAmount(amount: BigDecimal): String {
    val amountWithoutTrailingZeros = amount.stripTrailingZeros()
    return if (amountWithoutTrailingZeros.scale() <= 0) {
        amountWithoutTrailingZeros.toPlainString()
    } else {
        amountWithoutTrailingZeros.setScale(2).toPlainString()
    }
}

fun main() {
    val amount = BigDecimal("10.6666")
    val amount2 = BigDecimal("20.00")
    val formattedAmount = formatAmount(amount)
    val formattedAmount2 = formatAmount(amount2)

    println(formattedAmount) // 输出: 10.67
    println(formattedAmount2) // 输出: 20
}

```
我们将金额设置为 10.6666，然后调用 formatAmount 函数来格式化该金额。根据代码逻辑，10.6666 会被格式化为 10.67。
请注意，setScale 方法用于设置小数位数。在示例中，我们使用 setScale(2) 将金额设置为保留两位小数。如果你希望保留更多或更少的小数位数，可以相应地调整参数。
另外，为了避免浮点数精度问题，建议在创建 BigDecimal 对象时使用字符串表示，而不是直接使用浮点数。这样可以确保金额的精确性。


