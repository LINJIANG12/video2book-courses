# Tabs Content 卡片头部的医生信息制作

卡片架子上一讲焊死了，这一讲我们填最要紧的一块：卡片头部。你们去看任何问诊 App，卡片第一眼就是“医生头像 + 名字 + 职称 + 关注按钮”，这一行要是歪了，用户对整个页面的信任感直接掉一半。所以千万别小看这几十像素，我带你们一行一行把它对齐。

## 1. 头部三段式的盒子划分

头部看着元素多，其实就三段：左边头像，中间姓名职称，右边关注按钮。用弹性盒子（Flexible Box，`display: flex`）一排摆开，中间那段自动撑满：

```text
+--------------------------------------------------+
| [头像]  王医生  主任医师  三甲医院        [+关注] |
|  50px   flex:1 (自动撑满, 垂直居中)       定宽按钮 |
+--------------------------------------------------+
```

读图说明：头像定宽定高，按钮定宽，中间文字区 `flex: 1` 把剩余空间全吃掉，这样换多长的名字都不会把按钮挤掉。

结构：

```html
<div class="card-head">
  <img class="avatar" src="./assets/images/doctor.png" alt="医生头像" />
  <div class="doc-info">
    <h4>王小医 <span>主任医师</span></h4>
    <p>北京协和医院 · 儿科</p>
  </div>
  <a class="follow" href="#">+ 关注</a>
</div>
```

你们看，名字用 `h4`（标题语义），职称和医院用 `p` 和 `span`（说明语义），语义（Semantics）顺手就带上了。不要满屏全用 `div`，搜索引擎（Search Engine Optimization，SEO）和读屏软件都靠这些标签认内容。

## 2. 头像与文字的对齐写法

```css
.card-head {
  display: flex;
  align-items: center;
}
.card-head .avatar {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  object-fit: cover;
}
.doc-info {
  flex: 1;
  margin-left: 10px;
  overflow: hidden;
}
.doc-info h4 {
  font-size: 15px;
  color: #333;
}
.doc-info h4 span {
  margin-left: 6px;
  font-size: 12px;
  font-weight: 400;
  color: #999;
}
.doc-info p {
  margin-top: 4px;
  font-size: 12px;
  color: #999;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

这里有三个细节你们一定要抠。第一，头像必须正圆，`border-radius: 50%` 配等宽等高，少一边都不圆；第二，`object-fit: cover` 让长方形原图按比例裁成圆脸，不加这句人物会被拉变形；第三，中间文字区加 `overflow: hidden`，医院名字再长也不会把布局撑爆，而是乖乖省略。

> **易错点**：`align-items: center` 写成 `justify-content: center` 的人最多。记住，主轴（Main Axis）默认是横向，`justify-content` 管横向的左右分布，`align-items` 管交叉轴（Cross Axis）的上下对齐。头像文字上下居中，一定是 `align-items` 的事。

## 3. 右侧关注按钮的做法

```css
.card-head .follow {
  flex: none;
  padding: 6px 12px;
  font-size: 12px;
  color: #16c2a3;
  border: 1px solid #16c2a3;
  border-radius: 15px;
}
```

描边按钮（Outline Button）三要素：文字色、边框色、圆角一次写全。圆角给 `15px` 这种超过高度一半的值，按钮两头就是标准的胶囊形。后面点击变成“已关注”的灰态，我们做交互时再切换类名，结构不用动。

> **结论**：卡片头部就是“头像定尺寸、中间自动撑、按钮不收缩”九个字。你们自查：头像圆不圆、名字职称一行还是两行、医院太长有没有省略、按钮有没有被挤扁。这四处都对，这一讲就算过关，下一讲我们做卡片底部。
