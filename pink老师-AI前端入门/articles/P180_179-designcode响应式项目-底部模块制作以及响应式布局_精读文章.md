# 底部模块制作与响应式布局

终于到最后一块了，底部（footer）模块。你们别小看页脚，用户滑到这里，意味着他把整页都看完了，肯不肯点个订阅、关不关注社交账号，全看这一屏。我跟你们说，页脚拼的是信息分组：左边品牌加订阅，中间几列链接，底部一行版权加社交图标，分组清楚，一点都不乱。

这一讲我们两件事一起干：先把四列结构搭出来，再把响应式收尾。到手机上四列变一列，订阅框顶满，链接两两并排，这就是标准答案。

## 1 页脚四列结构搭建

我用的还是语义化标签，外层 `footer`，里面 `.footer-top` 放四列，`.footer-bottom` 放版权行。四列分别是品牌订阅、课程链接、公司链接、资源链接。订阅那一列最宽，给 `1.4fr`，后面三列各 `1fr`。

```html
<footer class="footer">
  <div class="footer-top">
    <div class="brand">
      <img src="./images/logo.png" alt="网站标志" />
      <p>从零开始，做出能上线的响应式页面。</p>
      <form class="subscribe">
        <input type="email" placeholder="输入你的邮箱" />
        <button type="submit">订阅</button>
      </form>
    </div>
    <dl class="links">
      <dt>课程</dt>
      <dd><a href="#">前端入门</a></dd>
      <dd><a href="#">响应式布局</a></dd>
      <dd><a href="#">实战项目</a></dd>
    </dl>
    <dl class="links">
      <dt>公司</dt>
      <dd><a href="#">关于我们</a></dd>
      <dd><a href="#">加入我们</a></dd>
      <dd><a href="#">联系方式</a></dd>
    </dl>
    <dl class="links">
      <dt>资源</dt>
      <dd><a href="#">帮助中心</a></dd>
      <dd><a href="#">社区</a></dd>
      <dd><a href="#">更新日志</a></dd>
    </dl>
  </div>
  <div class="footer-bottom">
    <p>© 2025 DesignCode · 保留所有权利</p>
    <div class="social"><a href="#">B站</a><a href="#">微博</a><a href="#">微信</a></div>
  </div>
</footer>
```

> **提示**：链接组我用 `dl` 加 `dt` 加 `dd` 写，标题是 `dt`，链接是 `dd`。这比全用 `div` 加 `a` 语义准得多，读屏软件也能直接读出分组关系。

订阅表单那一行，输入框和按钮我包在一个圆角胶囊里，输入框去掉边框、背景透明，按钮给深色底。你们记住这一条就够了，页脚的订阅框永远做成胶囊形，用户一眼就知道这是要填的。

```css
.footer-top {
  display: grid;
  grid-template-columns: 1.4fr 1fr 1fr 1fr;
  gap: 32px;
}
.subscribe {
  display: flex;
  align-items: center;
  background-color: #f3f4f6;
  border-radius: 999px;
  padding: 6px 6px 6px 18px;
}
.subscribe input {
  flex: 1;
  border: 0;
  outline: none;
  background-color: transparent;
}
.subscribe button {
  border: 0;
  border-radius: 999px;
  padding: 10px 22px;
  background-color: #111827;
  color: #fff;
  cursor: pointer;
}
```

整体分块关系是这样的：

```text
.footer
  |-- .footer-top [grid 4列]
  |     |-- .brand (logo + 简介 + 订阅胶囊) 1.4fr
  |     |-- .links 课程  1fr
  |     |-- .links 公司  1fr
  |     |-- .links 资源  1fr
  |-- .footer-bottom [flex 左右]
        |-- 版权文字  +  社交链接
```

读图说明：顶部四列中品牌列最宽含订阅表单，三组链接等宽并排，底部一行左右分布版权与社交入口。

## 2 页脚响应式布局

到 `1024px` 以下，四列先收成两列，品牌独占第一行通栏，后面三组链接三变两加一，刚好两行。到 `768px` 以下，全部单列，链接组改成两列小网格，省高度。

```css
@media (max-width: 1024px) {
  .footer-top {
    grid-template-columns: 1fr 1fr;
  }
  .brand {
    grid-column: 1 / -1;
  }
}
@media (max-width: 768px) {
  .footer-top {
    grid-template-columns: 1fr;
  }
  .footer-bottom {
    flex-direction: column;
    gap: 12px;
    text-align: center;
  }
  .subscribe {
    width: 100%;
  }
}
```

> **易错点**：`grid-column: 1 / -1` 是让品牌列横跨整行，从第一条网格线直接跨到最后一条。很多同学在这里写 `span 2`，列数一变就错位，写 `1 / -1` 不管几列都通吃。

| 宽度区间 | 顶部排法 | 底部排法 |
| :--- | :--- | :--- |
| 大于 1024px | 四列，品牌 1.4fr | 左右分布 |
| 768px 到 1024px | 两列，品牌通栏 | 左右分布 |
| 小于 768px | 单列堆叠 | 上下居中堆叠 |

订阅框到手机上一定要给 `width: 100%`，不然它缩在左上角，右边空一大截，特别难看。按钮的字也别太多，订阅两个字就够了，写立即订阅四个字，窄屏下输入框会被挤得只剩一半。

做到这里，DesignCode 从顶栏、导航、课程卡、故事卡、波浪到页脚，整条线就串起来了。你们回头从 1400px 拖到 360px 完整走一遍，没有横向滚动、没有图片变形、hover 进出都有动画，这个项目就算真正过关了。
