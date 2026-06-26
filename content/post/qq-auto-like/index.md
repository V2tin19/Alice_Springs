---
title: 关于我写了一个QQ自动回赞脚本这件事
description: 大号被封，小号没资历，自己动手刷赞
slug: qq-auto-like
date: 2026-06-26 20:11:56+0000
image: cover.jpg
categories:
    - 折腾
tags:
    - Auto.js
    - QQ
    - 自动化
weight: 1
---

前段时间我的QQ大号被封了，小号等级又太低，一进群就被人说是小号，很没面子。但等级这东西一时半会儿也升不上去，怎么办？我想到了另一个指标——**赞数**。只要主页赞数够多，看起来就像个老用户，资历一下就撑起来了。

去淘宝搜了一下刷赞，要钱。而且很多是机器刷的，容易被检测。想了想，不如自己动手，丰衣足食。

## 思路

QQ空间里有个互赞功能，大家互相点头像主页的赞。我加了几个互赞群，发现里面回赞率其实挺高的——大多是小孩和不知道到底上不上班的大人，你给他点了，他真回你。

问题只有一个：手动点太烦了。一个群几百人，一个个点过去，手都点麻了。所以我写了个 Auto.js 脚本，自动翻页、自动点赞、一直点到没人为止。

## 代码

```javascript
"auto";

// ========== 配置 ==========
var CLICKS_PER_FRIEND = 20;     // 每个好友点击次数
var CLICK_DELAY = 5;            // 点击间隔（毫秒），5ms 已经极快，可尝试 0
var SCROLL_WAIT = 200;          // 翻页后等待（毫秒），200ms 足够大多数情况

// ========== 引擎控制 ==========
var currentEngine = engines.myEngine();
function isStopped() { return currentEngine.isDestroyed(); }

console.show();
log("启动，每好友 " + CLICKS_PER_FRIEND + " 下，延迟 " + CLICK_DELAY + "ms");

// 查找赞按钮（完全不变）
function findAllLikeButtons() {
    var candidates = [];
    var byText = text("赞").find();
    if (byText.length > 0) candidates = candidates.concat(byText);
    var byDesc = desc("赞").find();
    if (byDesc.length > 0) candidates = candidates.concat(byDesc);
    var byDesc2 = desc("点赞").find();
    if (byDesc2.length > 0) candidates = candidates.concat(byDesc2);
    if (candidates.length === 0) {
        var images = className("ImageView").find();
        for (var i = 0; i < images.length; i++) {
            var v = images[i];
            var d = v.desc();
            if (d && d.indexOf("赞") !== -1) candidates.push(v);
        }
    }
    var unique = [];
    var coordSet = {};
    for (var i = 0; i < candidates.length; i++) {
        var b = candidates[i].bounds();
        if (!b) continue;
        var key = b.centerX() + "," + b.centerY();
        if (!coordSet[key]) {
            coordSet[key] = true;
            unique.push(candidates[i]);
        }
    }
    return unique;
}

// 翻页
function scrollNextPage() {
    var list = className("AbsListView").scrollable().findOne(500);
    if (list) {
        list.scrollForward();
    } else {
        swipe(device.width / 2, device.height * 0.7, device.width / 2, device.height * 0.3, 300);
    }
    sleep(SCROLL_WAIT);
}

// 主程序
toast("请打开互赞页面");
waitForActivity("com.tencent.mobileqq.activity.VisitorsActivity", 10000);

var emptyPageCount = 0;

while (!isStopped()) {
    var buttons = findAllLikeButtons();

    if (buttons.length === 0) {
        emptyPageCount++;
        if (emptyPageCount > 5) break;
        scrollNextPage();
        continue;
    }
    emptyPageCount = 0;

    // 预取所有坐标，减少循环内属性访问
    var coords = [];
    for (var i = 0; i < buttons.length; i++) {
        var b = buttons[i].bounds();
        if (b) coords.push({ x: b.centerX(), y: b.centerY() });
    }

    log("本页 " + coords.length + " 人");

    for (var i = 0; i < coords.length; i++) {
        if (isStopped()) break;
        var x = coords[i].x, y = coords[i].y;
        for (var j = 0; j < CLICKS_PER_FRIEND; j++) {
            if (isStopped()) break;
            click(x, y);
            if (CLICK_DELAY > 0) sleep(CLICK_DELAY);
        }
    }

    if (isStopped()) break;
    log("翻页");
    scrollNextPage();
}

log("结束");
toast("脚本已结束");
```

## 使用方法

1. 安装 Auto.js（需要开启无障碍服务权限）。
2. 把上面的代码粘贴进去，保存运行。
3. **关键步骤**：先打开自己的 QQ 主页，停顿一秒，再进入互赞页面。
4. 运行脚本，让它自己翻页、自己点。

## 一些注意事项

- **5ms 的点击间隔**手机不一定反应得过来，实际速度可能没你想的那么快。如果手机卡，可以调大一点。
- **翻页时会重复赞交界的人**。因为 QQ 的列表滑动不是完全对齐的，上一页最后一个人可能还在下一页第一行，脚本会再点一遍。无伤大雅，多赞一下不算亏。
- **脚本在赞完之前没有主动停止的能力**。你让它停，它只有等到当前这页点完、翻页检测时才会停。急的话直接关掉 Auto.js。
- **没有主动点击"查看更多"的能力**。如果列表加载更多需要手动点按钮，脚本只会翻页，不会点那个按钮。所以最好选那种无限下滑的互赞群。

## 效果

第一天多找几个诚信真实的网友，大多数是小孩和不知道到底上不上班的大人，回赞率都挺高。一天靠脚本把赞数刷完后，能拿到几百的回赞。一个星期下来，两千三千赞是有的。

非常好用。现在我的小号主页看起来像个老江湖了。

---

*免责声明：脚本仅供学习交流，请勿用于恶意刷量或违反平台规则的行为。被封号别找我。*

---

*本文由 Kimi 大模型辅助写作并上传。*
