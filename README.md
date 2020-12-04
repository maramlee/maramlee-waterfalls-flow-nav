# maramlee-waterfalls-flow-nav 使用

## 前言

maramlee-waterfalls-flow-nav 是基于 maramlee-waterfalls-flow 为了解决很多小伙伴儿实际场景使用瀑布流需要使用 nav 切换数据而封装的插件。没错，使用这个插件，记得先安装 [maramlee-waterfalls-flow](https://ext.dcloud.net.cn/plugin?id=2714#rating) 插件。当然，除了 maramlee-waterfalls-flow 插件，还有需安装 [uni-load-more](https://ext.dcloud.net.cn/plugin?id=29)（加载更多） 插件。

## 使用方式

我的插件代码中有详细的说明，如果想要了解更多，请查看插件代码中的注释。

### 首先在 `script` 中定义数据和方法等

```javascript
import waterfallsFlowNav from "@/components/maramlee-waterfalls-flow-nav/maramlee-waterfalls-flow-nav.vue";
export default {
  components: { waterfallsFlowNav },
  data() {
    return {
      navIndex: 0, // 默认获取的第几项数据
      navData: [
        /**
         * nav 对应的数据
         * 注意插件里面下面两个 prop 值的设置：
         *   |- list_key: { type: String, default: "key" },
         *   |- list_label: { type: String, default: "label" },
         */
        { key: "one", label: "nav 1" },
        { key: "two", label: "nav 2" },
        { key: "three", label: "nav 3" },
      ],
    };
  },
  /**
   * 上拉加载 这很重要
   */
  onReachBottom() {
    /**
     * 这里的 waterfalls_flow_nav 值记得与你定义的 ref 值对应
     */
    this.$refs.waterfalls_flow_nav.getList();
  },

  methods: {
    /**
     * 此方法为 视图层响应子组件 add-data 事件 的方法
     */
    getListHandle() {
      /**
       * mockData 为模拟数据
       * 实际应为接口返回的数据
       */
      const mockData = {
        total_page: 1,
        list: [
          {
            id: 1,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599475741266&di=e36d6c01c93320e2ba1504d8357248f4&imgtype=0&src=http%3A%2F%2Fa0.att.hudong.com%2F30%2F29%2F01300000201438121627296084016.jpg",
            title: "可爱的小猫咪呀",
            text:
              "小小的猫咪，甚是呆萌，呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌",
          },
        ],
      };
      /**
       * ================= !important =================
       * 模拟请求成功
       * 实际场景中是 request 请求哈
       */
      setTimeout(() => {
        this.$refs.waterfalls_flow_nav.successSetData(
          mockData.list,
          mockData.total_page
        );
      }, 1000);
      /**
       * ================= !important =================
       * 若失败，记得调用失败回调方法
       * 此例如下：
       *   this.$refs.waterfalls_flow_nav.failMoreBack();
       */
    },
  },
};
```

### 再在 `template` 中使用组件

注意，下面的数据使用，都是基于上一条的数据定义，记得对照着看。

#### 可以只是渲染图片，不需要其他

```vue
<template>
  <waterfallsFlowNav
    ref="waterfalls_flow_nav"
    v-model="navIndex"
    :navData="navData"
    :single="true"
    @add-data="getListHandle"
  />
</template>
```

#### 有插槽（自定义内容）的情况要分情况使用

##### 若只适配 app、h5 端，利用作用域插槽

注意：`item` 包含 `list` 对应的数据项，可以随意搭配、自定义使用。

```vue
<template>
  <waterfallsFlowNav
    ref="waterfalls_flow_nav"
    v-model="navIndex"
    :navData="navData"
    :single="true"
    @add-data="getListHandle"
  >
    <template v-slot:default="item">
      <!-- 此处添加插槽内容 -->
      <!-- <view class="cnt">
          <view class="title">{{item.title}}</view>
          <view class="text">{{item.text}}</view>
        </view> -->
    </template>
  </waterfallsFlowNav>
</template>
```

##### 若只适配微信小程序，利用小程序插槽

注意：微信小程序没有动态模板，也和 vue 的插槽使用方式不一样。

由于小程序的复杂性，又不想数据提到外层，我在插件里面定义了一个 weixin_type prop ，用于小程序不同模板控制。小程序使用时，需使用者自己去配置 maramlee-waterfalls-flow-nav 插件的模板内容。

小程序使用步骤如下：

1. 第一步：页面中 template 应用插件

```vue
<template>
  <waterfallsFlowNav
    ref="waterfalls_flow_nav"
    v-model="navIndex"
    :navData="navData"
    @add-data="getListHandle"
  />
</template>
```

2. 第二步：修改 maramlee-waterfalls-flow-nav 插件图片下方模板

注意：这里修改的是插件 components/maramlee-waterfalls-flow-nav/maramlee-waterfalls-flow-nav.vue

如插件中 53 行注释的类似，script 标签中 116-126 行有详细注释，请自行查看。

```vue
<!--
  取自 maramlee-waterfalls-flow-nav 组件中 36-56 行代码
 -->
<!--  #ifdef  MP-WEIXIN -->
<template v-if="!single">
  <view v-for="(item, index2) of obj.list" :key="index2" slot="slot{{index2}}">
    <view v-if="weixin_type === 'default'" class="cnt">
      <view class="title">{{ item.title }}</view>
      <view class="text">{{ item.text }}</view>
    </view>
    <!-- 
          ==========================================
          weixin_type 为 "two" 时样例代码新增示例
          用时记得取消注释
          ==========================================
         -->
    <!-- <view v-else-if="weixin_type === 'two'">我是类型为 two 的样子哟</view> -->
  </view>
</template>
<!--  #endif -->
```

```javascript
// 取自 maramlee-waterfalls-flow-nav 组件 116-133 行代码
/**
 * =================================================
 * 微信小程序特殊自定义不同类型下方内容
 * 默认为 "default"
 * 使用示例，例如新增一个 "two" 类型：
 *   1.修改 weixin_type 的 validator：
 *     |- validator: (value) => ["default","two"].indexOf(value) !== -1,
 *
 *   2. 上方代码块处新增类型为 two 的结构代码，详情请看上方代码处
 * =================================================
 */
// #ifdef MP-WEIXIN
weixin_type: {
  type: String,
  default: "default",
  validator: (value) => ["default"].indexOf(value) !== -1,
},
// #endif
```

##### 若需要同时兼容 app、h5、微信小程序，则需条件渲染

1. 首先使用 适配 app、h5 的方式模板。

```vue
<template>
  <waterfallsFlowNav
    ref="waterfalls_flow_nav"
    v-model="navIndex"
    :navData="navData"
    :single="true"
    @add-data="getListHandle"
  >
    <template v-slot:default="item">
      <!-- 此处添加插槽内容 -->
      <!-- <view class="cnt">
          <view class="title">{{item.title}}</view>
          <view class="text">{{item.text}}</view>
        </view> -->
    </template>
  </waterfallsFlowNav>
</template>
```

2. 再与上一条小程序适配一样的方式适配即可，这里不再赘述。

## 属性说明

<!-- navIndex: { type: Number, required: true },
    navData: { type: Array, required: true },

    /**
     * nav 渲染 key
     * 注意查看我的示例里
     * navData 数据格式与之对应：[{ key: "one", label: "nav 1" }]
     */
    list_key: { type: String, default: "key" },
    list_label: { type: String, default: "label" },
    // mounted 时，是否请求数据，默认请求
    mountedGet: { type: Boolean, default: true }, -->

| 属性名      | 类型    | 默认值    | 是否必传 | 说明                                                        | 平台支持         |
| ----------- | ------- | --------- | -------- | ----------------------------------------------------------- | ---------------- |
| navData     | Array   | -         | 是       | 渲染 nav 的列表，一般格式为：[{ key: "one", label: "nav" }] | 全               |
| navIndex    | Number  | -         | 是       | nav 高亮的 index，一般页面中设置 0                          | 全               |
| list_key    | String  | key       | 否       | 与 navData 项的 key 对应                                    | 全               |
| list_label  | String  | label     | 否       | 与 navData 项的 label 对应                                  | 全               |
| mountedGet  | Boolean | true      | 否       | 插件 mounted 时是否请求数据                                 | 全               |
| offset      | Number  | 10        | 否       | 单位是 px                                                   | 全               |
| idKey       | String  | id        | 否       | 列表渲染的 key 的键名，值必须唯一                           | 全               |
| imageSrcKey | String  | image_url | 否       | 图片 src 的键名                                             | 全               |
| cols        | Number  | 2         | 否       | 列数，值必须不小于 2                                        | 全               |
| single      | Boolean | false     | 否       | 是否是单独的渲染图片，只控制图片圆角而已                    | 全               |
| listStyle   | Object  | -         | 否       | 单个展示项的样式                                            | 微信小程序不支持 |
| imageStyle  | Object  | -         | 否       | 图片的样式                                                  | 全               |

## 事件说明

| 事件名       | 说明                           | 返回值           |
| ------------ | ------------------------------ | ---------------- |
| @add-data    | 加载数据事件，很重要的一个事件 | 无               |
| @nav-click   | nav 点击时触发                 | 点击项对应 index |
| @wapper-lick | 单项点击事件                   | 单项对应数据     |
| @image-click | 图片点击事件                   | 单项对应数据     |
| @image-load  | 所有图片渲染完成触发           | -                |

### add-data 事件详解

此事件为解决不同的人封装的请求方法不同，不同接口请求的数据不同而产生。必需要配合很重要的两个组件方法：successSetData、failMoreBack 使用。

```javascript
getListHandle() {
  const mockData = {
    total_page: 1,
    list: [
      {
        id: 1,
        image_url:
          "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599475741266&di=e36d6c01c93320e2ba1504d8357248f4&imgtype=0&src=http%3A%2F%2Fa0.att.hudong.com%2F30%2F29%2F01300000201438121627296084016.jpg",
        title: "可爱的小猫咪呀",
        text:
          "小小的猫咪，甚是呆萌，呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌",
      },
    ],
  };
  /**
   * ================= !important =================
   * 模拟请求成功
   * 实际场景中是 request 请求哈
   */
  setTimeout(() => {
    this.$refs.waterfalls_flow_nav.successSetData(
      mockData.list,
      mockData.total_page
    );
  }, 1000);
  /**
   * ================= !important =================
   * 若失败，记得调用失败回调方法
   * 此例如下：
   *   this.$refs.waterfalls_flow_nav.failMoreBack();
   */
},
```

## 组件方法

| 方法名         | 说明                           | 参数                | 使用场景                            |
| -------------- | ------------------------------ | ------------------- | ----------------------------------- |
| getList        | 获取列表                       | isRefresh, navClick | 数据获取                            |
| refresh        | 刷新对应的 moreNavIndex 数据   | -                   | 下拉刷新                            |
| successSetData | 数据请求获取成功后设置组件数据 | list, total_page    | add-data 事件中，数据请求成功后调用 |
| failMoreBack   | 数据请求获取失败后设置组件数据 | -                   | add-data 事件中，数据请求失败后调用 |
| initNavLists   | 初始化插件内 moreNavLists 数据 | -                   | 使用情况比较少，需要时即调用        |

### getList 方法说明

获取列表方法

this.$refs.waterfalls_flow_nav.getList();

#### 参数

1. isRefresh

   默认 false

   是否强制刷新，若为 true，即请求之前先把 moreNavIndex 对应的原来的数据清空，常用在下拉刷新

2. navClick

   默认 false

   是否是 nav 点击，此值区分一般的加载和点击 nav 加载，由于 nav 点击，如果已经有数据，即无需加载数据 \*

#### 用法示例

1. 页面中用于上拉加载更多（onReachBottom）中：this.$refs.waterfalls_flow_nav.getList();
2. 页面中用于重新加载此项：this.$refs.waterfalls_flow_nav.getList(true);

### successSetData 使用

**注意查看 add-data 事件详解**

请求数据成功后调用

this.$refs.waterfalls_flow_nav.successSetData(list, total_page);

#### 参数

1. list

   必传

   请求返回需要渲染的 list 数据

2. total_page

   必传

   请求返回的总页数

### initNavLists 与 refresh 比较

initNavLists 是初始化 moreNavLists 的数据，即所有的数据清空。

refresh 只清空 moreNavIndex 对应即当前选择的 nav 项的数据。

## 提示

如果你看到了这里，还有以下情况：

1. 插件使用方法有不懂的地方；
2. 插件本身研究不明白的地方；
3. 觉得插件有待提高的建议；
4. 或者其他你遇到的问题；
5. 纯粹想前端技术交流也行。

可以加我微信，微信号：`ml-maramlee`，备注以`ml-${姓名}-${1}`的形式，其中 1、2、3、4 对应前面的情况（前端都看得懂哈），例如：`ml-maram-1`。

**注：人家是有脾气的，不这样备注不给加哟~**

**再注：觉得好用记得收藏、评论，这样可以让更多人看到，让更多人受益。当然，解答不易，欢迎赞赏。**

**申明：主要是看到评论里有或多或少的问题，加微信有助于解决问题，只接受技术交流，其他请勿扰。**

## 使用样例

pages/nav/nav.vue

```vue
<template>
  <view class="content">
    <waterfallsFlowNav
      ref="waterfalls_flow_nav"
      v-model="navIndex"
      :navData="navData"
      @add-data="getListHandle"
    >
      <template v-slot:default="item">
        <view class="cnt">
          <view class="title">{{ item.title }}</view>
          <view class="text">{{ item.text }}</view>
        </view>
      </template>
    </waterfallsFlowNav>
  </view>
</template>
<script>
import waterfallsFlowNav from "@/components/maramlee-waterfalls-flow-nav/maramlee-waterfalls-flow-nav.vue";
export default {
  components: { waterfallsFlowNav },
  data() {
    return {
      navIndex: 0,
      navData: [
        { key: "one", label: "nav 1" },
        { key: "two", label: "nav 2" },
        { key: "three", label: "nav 3" },
      ],
    };
  },
  /**
   * 上拉加载
   * 这很重要
   */
  onReachBottom() {
    this.$refs.waterfalls_flow_nav.getList();
  },
  methods: {
    getListHandle() {
      const mockData = {
        total_page: 1,
        list: [
          {
            id: 1,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599475741266&di=e36d6c01c93320e2ba1504d8357248f4&imgtype=0&src=http%3A%2F%2Fa0.att.hudong.com%2F30%2F29%2F01300000201438121627296084016.jpg",
            title: "可爱的小猫咪呀",
            text:
              "小小的猫咪，甚是呆萌，呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌呆萌",
          },
          {
            id: 2,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599475934834&di=7a37b8d628252c4aced6ed0decba9442&imgtype=0&src=http%3A%2F%2Fa3.att.hudong.com%2F43%2F74%2F01300000164151121808741085971.jpg",
            title: "迪士尼动画",
            text: "迪士尼动画之……",
          },
          {
            id: 3,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599476083909&di=a5debff35edec5de105bc105d6fdbce3&imgtype=0&src=http%3A%2F%2Fa2.att.hudong.com%2F77%2F77%2F01300000336597125202779973172.jpg",
            title: "火箭",
            text: "火箭升空瞬间，宏伟壮观啊",
          },
          {
            id: 5,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599476129760&di=7a3db0b14f6a74240bbfa7922ba22f45&imgtype=0&src=http%3A%2F%2Fa4.att.hudong.com%2F82%2F55%2F01300000349330124003555691086.jpg",
            title: "华佗",
            text: "华佗人物画像 中国画 线条画 毛笔画 彩色画",
          },
          {
            id: 6,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599476215687&di=97c2bbf6f3a1a3e2a6a2dc77dfe4bea7&imgtype=0&src=http%3A%2F%2Fa4.att.hudong.com%2F72%2F82%2F19300000009075130804824786610.jpg",
            title: "恐龙",
            text: "恐龙来啦",
          },
          {
            id: 7,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599476258176&di=29622b0f0cfd659aecebabaae352d02c&imgtype=0&src=http%3A%2F%2F1882.img.pp.sohu.com.cn%2Fimages%2Fblog%2F2011%2F3%2F25%2F13%2F13%2Fu48513077_12fa4ba953ag213.jpg",
            title: "手",
            text: "什么？",
          },
          {
            id: 8,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599476300222&di=49712f992d8f7bbb1a5851eced71cbe2&imgtype=0&src=http%3A%2F%2Fa2.att.hudong.com%2F71%2F56%2F16300000988660128426569668958.jpg",
            title: "百年好合",
            text: "百年好合 结婚 庚帖 二次元",
          },
          {
            id: 9,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599476416001&di=ea1a1f8f9b1274d39c05af3e48041e6a&imgtype=0&src=http%3A%2F%2Finews.gtimg.com%2Fnewsapp_bt%2F0%2F12420002963%2F641",
            title: "5G",
            text: "5G 来啦",
          },
          {
            id: 12,
            image_url:
              "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1599476567983&di=040976a1cd1a6e5510a237c57bdcff06&imgtype=0&src=http%3A%2F%2Finews.gtimg.com%2Fnewsapp_bt%2F0%2F12421051168%2F641",
            title: "王者荣耀",
            text: "王者荣耀 龙 快来打龙 请求集合",
          },
        ],
      };
      /**
       * ================= !important =================
       * 模拟请求成功
       * 实际场景中是 request 请求哈
       */
      setTimeout(() => {
        this.$refs.waterfalls_flow_nav.successSetData(
          mockData.list,
          mockData.total_page
        );
      }, 1000);
      /**
       * ================= !important =================
       * 若失败，记得调用失败回调方法
       * 此例如下：
       *   this.$refs.waterfalls_flow_nav.failMoreBack();
       */
    },
  },
};
</script>
<style>
page {
  background-color: #eee;
}
</style>
<style lang="scss" scoped>
.content {
  padding: 10px;
  .cnt {
    padding: 10px;
    .title {
      font-size: 16px;
    }
    .text {
      font-size: 14px;
      margin-top: 5px;
    }
  }
}
</style>
```
