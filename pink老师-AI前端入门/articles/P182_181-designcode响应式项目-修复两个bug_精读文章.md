# DesignCode 响应式项目的两个典型 Bug 修复

小圆飘起来之后我挺得意，拖着浏览器从大屏缩到小屏来回测，结果一下子测出两个毛病：一个是导航在小屏下切来切去状态全乱了，一个是页面底部莫名多出一截横向滚动条。你们做响应式肯定也撞过这种“桌面好好的，一缩就崩”的情况，这一讲我们就把这两个坑填平。

---

## 1. 导航切换状态残留问题

先说导航。我用的是经典做法：大屏显示横排菜单，小屏收成汉堡按钮，点击展开下拉。逻辑本身不难，难在窗口缩放时状态没同步。

复现路径很固定：先把窗口缩到 `768px` 以下，点汉堡把菜单打开，再把窗口拉回大屏，菜单就卡在展开的 mobile 样式上，横排全乱了。反过来也一样：大屏好好的，缩到小屏菜单却默认摊开着，把首屏顶下去一大截。

```text
状态残留复现路径
大屏(菜单横排) --缩小--> 小屏(菜单应收起, 实则摊开)
小屏(菜单展开) --放大--> 大屏(菜单应横排, 实则竖排残留)
```

读图说明：断点切换时若只靠点击事件控制显隐，不监听宽度变化，旧状态会带到新断点。

根因在我自己：我用 `JS` 给导航加了 `open` 类做展开收起，但媒体查询里没把这个类的影响对冲掉。修复要两端一起改。

```css
/* 默认 mobile 下菜单收起 */
.nav-list {
  display: none;
}
.nav-list.open {
  display: flex;
  flex-direction: column;
}
/* 回到大屏, 强制横排, 无视 open */
@media (min-width: 769px) {
  .nav-list {
    display: flex;
    flex-direction: row;
  }
}
```

```javascript
const toggleBtn = document.querySelector('.nav-toggle');
const navList = document.querySelector('.nav-list');

toggleBtn.addEventListener('click', () => {
  navList.classList.toggle('open');
});

// 关键补丁: 跨断点时清掉残留状态
window.addEventListener('resize', () => {
  if (window.innerWidth > 768 && navList.classList.contains('open')) {
    navList.classList.remove('open');
  }
});
```

> **易错点**：只写点击切换、不写 `resize` 复位，是响应式导航最常见的半截工程。一定要如此，绝不能那般：断点切换必须有状态兜底。

> **提示**：`resize` 里不要做重活，只做类名清理。菜单的显隐全部交给 `CSS`，`JS` 只负责换类，这样缩放时才不会闪。

改完再按刚才的路径测两遍：小屏点开再拉大，大屏缩到小，菜单都老老实实回到各自断点该有的样子，这个 `bug` 才算修死。

---

## 2. 装饰溢出导致的横向滚动条

第二个 `bug` 更隐蔽。页面在手机宽度下左右能轻微晃动，底部出现横向滚动条。我一开始还以为是某张图没压住，逐段删着查，最后定位到上一讲加的小圆和卡片区的装饰条：绝对定位的元素一半探出父盒子，把 `body` 撑宽了。

```text
溢出示意 (俯视宽度方向)
+-------- body 375px --------+
| .banner (overflow可见)     |
|   文字区                   |
|              (o) dot2 ->| 探出 20px
|                         |  ==> body 被撑到 395px
+----------------------------+
```

读图说明：探出父盒子的装饰部分会计入页面滚动宽度，手机上就会左右晃。

修复分两层。第一层是治标：给可能溢出的父级和 `body` 加保险。

```css
html, body {
  overflow-x: hidden;
}
.banner,
.cards {
  overflow: hidden;
  position: relative;
}
```

第二层是治本：小屏下把探出去的装饰收回来或藏掉，而不是靠裁剪硬盖。

```css
@media (max-width: 768px) {
  .dot2 {
    display: none;
  }
  .card-deco {
    right: 0;
    transform: none;
  }
  .cards img {
    max-width: 100%;
    height: auto;
  }
}
```

| 排查手段 | 发现的问题 | 修复归属 |
| :--- | :--- | :--- |
| 删段二分法定位晃动来源 | `dot2` 右探出约 `20px` | 小屏隐藏 + 父级裁剪 |
| 检查图片与卡片定宽 | 卡片 `width: 320px` 在 `320px` 屏下加边距溢出 | 改 `max-width: 100%` |
| 缩放复测 `375px / 360px` | 装饰条 `right: -30px` 探出 | 小屏归零 |

> **结论**：`overflow-x: hidden` 是保险丝，不是修法。真正要改的是溢出的那个元素本身，否则桌面端看着没事，换个更窄的机型又冒出来。

两个修完，我的验收动作是固定的，你们也照做：`1200px`、`768px`、`375px` 三个宽度各刷新一次，点一遍汉堡开合，再左右拖一下看有没有横向滚动。三遍全过，DesignCode 这个响应式项目才算收尾，后面我们进 `Bootstrap`，很多这类布局就不用手写了。
