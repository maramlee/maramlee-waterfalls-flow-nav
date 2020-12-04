<template>
  <view>
    <!-- 
      这里 nav 样式是随意写的
      可以自行提组件定义或修改
      重要的是要注意 navClickHandle 方法
     -->
    <view class="nav-nav">
      <view
        class="nav-nav-item"
        v-for="(item, index) of navData"
        :key="item[list_key]"
        :class="{ active: index === moreNavIndex }"
        @click="navClickHandle(item[list_key], index)"
        >{{ item[list_label] }}</view
      >
    </view>
    <waterfallsFlow
      v-show="index === moreNavIndex"
      v-for="(obj, index) of moreNavLists"
      :key="index"
      :list="obj.list"
      :ref="'waterfallsFlow-' + index"
      :idKey="idKey"
      :imageSrcKey="imageSrcKey"
      :single="single"
      @image-click="$emit('image-click', $event)"
      @wapper-click="$emit('wapper-click', $event)"
      @image-load="imageLoadHandle"
    >
      <!-- 
        微信小程序端
        此处小程序样式只做示例
        需要请自行修改
       -->
      <!--  #ifdef  MP-WEIXIN -->
      <template v-if="!single">
        <view
          v-for="(item, index2) of obj.list"
          :key="index2"
          slot="slot{{index2}}"
        >
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

      <!-- 非微信小程序端 -->
      <!-- #ifndef  MP-WEIXIN -->
      <template v-slot:default="item">
        <slot v-bind="{ ...item }" />
      </template>
      <!-- #endif -->
    </waterfallsFlow>
    <uni-load-more :status="moreNavLists[moreNavIndex].more" />
  </view>
</template>
<script>
import uniLoadMore from "@/components/uni-load-more/uni-load-more.vue";
import waterfallsFlow from "@/components/maramlee-waterfalls-flow/maramlee-waterfalls-flow.vue";

export default {
  components: { uniLoadMore, waterfallsFlow },
  data() {
    return {
      navActive: "",
      moreNavIndex: 0,
      moreNavLists: [],
      isLoading: false, // 是否正在加载数据中，控制重复加载、tab 切换问题
    };
  },
  model: { event: "nav-click", prop: "navIndex" },
  props: {
    navIndex: { type: Number, required: true },
    navData: { type: Array, required: true },

    /**
     * nav 渲染 key
     * 注意查看我的示例里
     * navData 数据格式与之对应：[{ key: "one", label: "nav 1" }]
     */
    list_key: { type: String, default: "key" },
    list_label: { type: String, default: "label" },
    // mounted 时，是否请求数据，默认请求
    mountedGet: { type: Boolean, default: true },

    /**
     * 下面参数给 maramlee-waterfalls-flow 提供
     */
    /**
     * ml 注：
     * 如果里面已经有了默认值
     *   |- app、h5 可以省略默认值
     *   |- 微信小程序却 ==|* 必须 *|== 写上默认值
     */
    offset: { type: Number, default: 10 },
    idKey: { type: String, default: "id" },
    imageSrcKey: { type: String, default: "image_url" },
    cols: { type: Number, default: 2, validator: (num) => num >= 2 },
    single: { type: Boolean, default: false },
    imageStyle: { type: Object },
    // #ifndef MP-WEIXIN
    listStyle: { type: Object },
    // #endif

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
  },
  watch: {
    navIndex() {
      this.moreNavIndex = this.navIndex;
    },
  },
  created() {
    this.moreNavIndex = this.navIndex;
    this.navActive = (this.navData[this.navIndex] || {}).key;
    this.initNavLists(this.navData);
  },
  mounted() {
    this.mountedGet && this.getList(true);
  },
  methods: {
    navClickHandle(key, index) {
      if (!this.isLoading) {
        this.moreNavIndex = index;
        this.$emit("nav-click", index);
        this.$nextTick(() => {
          this.getList(false, true);
        });
      }
    },
    /**
     * 页面布局渲染成功后设置参数 more 的回调
     * 设置 more 参数
     */
    imageLoadHandle() {
      this.$emit("image-load");
      const obj = this.moreNavLists[this.moreNavIndex];
      if (obj.page < obj.total_page) {
        obj.page += 1;
        obj.more = "more";
      } else obj.more = "noMore";
    },

    // 清空 moreNavIndex 数据，内部使用
    __moreRefresh() {
      const obj = this.moreNavLists[this.moreNavIndex];
      obj.list = [];
      obj.page = 1;
      obj.more = "more";
      obj.total_page = 0;
    },

    /**
     * ==*== ==*== ==*== ==*== ==*== ==*== ==*== ==*== ==*== ==*==
     * start
     * --*-- --*-- --*-- --*-- --*-- --*--
     * ml 注
     * 下方方法提供给外部组件调用
     * 使用方式统一为 this.$refs[waterfalls_flow_nav_name][method_handle]
     *   |- waterfalls_flow_nav_name 为您自定义的 ref 唯一标志
     *   |- method_handle 为下方方法
     * 不理解请参考我的实例
     * --*-- --*-- --*-- --*-- --*-- --*--
     */

    /**
     * 获取列表方法
     * @param isRefresh 是否强制刷新，若为 true，即请求之前先把 moreNavIndex 对应的原来的数据清空，常用在下拉刷新
     * @param navClick 是否是 nav 点击，此值区分一般的加载和点击 nav 加载，由于 nav 点击，如果已经有数据，即无需加载数据
     *
     * 用法示例：
     *   |- 页面中用于上拉加载更多（onReachBottom）中：this.$refs.waterfalls_flow_nav.getList();
     *   |- 页面中用于重新加载此项：this.$refs.waterfalls_flow_nav.getList(true);
     */
    getList(isRefresh = false, navClick = false) {
      if (!(navClick && this.moreNavLists[this.moreNavIndex].list.length)) {
        isRefresh && this.__moreRefresh();
        const obj = this.moreNavLists[this.moreNavIndex]; // 操作目标 对象
        if (obj.more === "more") {
          this.isLoading = true;
          obj.more = "loading";
          this.$emit("add-data");
        }
      }
    },
    /**
     * 刷新 moreNavIndex 当前项数据
     * 用于下拉刷新等
     */
    refresh() {
      this.__moreRefresh();
      this.$refs[`waterfallsFlow-${this.moreNavIndex}`][0].refresh();
    },

    /**
     * ================= !important =================
     * 请求数据成功后调用
     * 用于添加数据
     * @param list  必传 请求返回需要渲染的 list 数据
     * @param total_page 必传 请求返回的总页数
     */
    successSetData(list, total_page) {
      const obj = this.moreNavLists[this.moreNavIndex];
      obj.list = obj.list.concat(list);
      obj.total_page = total_page;
      this.isLoading = false;
      !obj.list.length && this.imageLoadHandle();
    },
    /**
     * ================= !important =================
     * 数据请求失败后调用
     */
    failMoreBack() {
      const obj = this.moreNavLists[this.moreNavIndex];
      obj.more = "more";
      this.isLoading = false;
    },
    /**
     * 初始化 moreNavLists
     * 使用场景很少，需要的时候，自然知道要用
     */
    initNavLists(navList) {
      let arr = [];
      navList.forEach(() => {
        arr.push({
          list: [], // 列表
          page: 1, // 当前页页数
          more: "more", // 是否继续加载
          total_page: 0,
        });
      });
      this.moreNavLists = arr;
    },
    /**
     * --*-- --*-- --*-- --*-- --*-- --*--
     * end
     * ==*== ==*== ==*== ==*== ==*== ==*== ==*== ==*== ==*== ==*==
     */
  },
};
</script>
<style lang="scss" scoped>
// 此样式仅供参考
// 样式请自行定义
.nav-nav {
  display: flex;
  .nav-nav-item {
    margin: 10px;
    padding-bottom: 3px;
    border-bottom: 2px solid transparent;
    &.active {
      color: #fd4604;
      border-bottom-color: #fd4604;
    }
  }
}

// 图片下方内容样式，小程序可自行定义
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
</style>
