<template>
  <view>
    <view class="nav-nav">
      <view
        v-for="(item, index) of navData"
        :key="item[list_key]"
        :class="{ active: index === moreNavIndex }"
        @click="navClickHandle(item[list_key], index)"
        >{{ item[list_label] }}</view
      >
    </view>
    <template v-if="moreNavLists.length">
      <waterfallsFlow
        v-show="index === moreNavIndex"
        v-for="(obj, index) of moreNavLists"
        :key="index"
        :list="(obj || {}).list"
        :ref="'waterfallsFlow-' + index"
        :idKey="idKey"
        :imageSrcKey="imageSrcKey"
        @image-click="$emit('image-click', $event)"
        @wapper-click="$emit('wapper-click', $event)"
        @image-load="imageLoadHandle"
      >
        <!--  #ifdef  MP-WEIXIN -->
        <!-- <view
        v-for="(item, index) of (obj || {}).list"
        :key="index"
        slot="slot{{index}}"
      >
        <view>jfjp</view>
      </view> -->
        <!-- <view>jfaojp</view> -->
        <!--  #endif -->

        <!-- #ifndef  MP-WEIXIN -->
        <template v-slot:default="item">
          <slot v-bind="{ ...item }" />
        </template>
        <!-- #endif -->
      </waterfallsFlow>
      {{ (moreNavLists[moreNavIndex] || {}).more }}
      <!-- <uni-load-more :status="'more'" /> -->
      <uni-load-more :status="(moreNavLists[moreNavIndex] || {}).more" />
    </template>
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

    // nav 渲染 key
    list_key: { type: String, default: "key" },
    list_label: { type: String, default: "label" },
    // mounted 时，是否请求数据，默认请求
    mountedGet: { type: Boolean, default: true },

    // 下面参数给 maramlee-waterfalls-flow 提供
    offset: { type: Number },
    idKey: { type: String },
    imageSrcKey: { type: String },
    cols: { type: Number, default: 2, validator: (num) => num >= 2 },
    imageStyle: { type: Object },
    single: { type: Boolean },
    // #ifndef MP-WEIXIN
    listStyle: { type: Object },
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
    this.moreNavLists = this.__initNavLists(this.navData);
  },
  mounted() {
    this.mountedGet && this.getList(true);
  },
  methods: {
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
      const obj = this.moreNavLists[this.moreNavIndex];
      if (obj.page < obj.total_page) {
        obj.page += 1;
        obj.more = "more";
      } else obj.more = "noMore";
    },

    __initNavLists(navList) {
      let arr = [];
      navList.forEach(() => {
        arr.push({
          list: [], // 列表
          page: 1, // 当前页页数
          more: "more", // 是否继续加载
          total_page: 0,
        });
      });
      return arr;
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
  & > * {
    margin: 10px;
    padding-bottom: 3px;
    border-bottom: 2px solid transparent;
    &.active {
      color: #fd4604;
      border-bottom-color: #fd4604;
    }
  }
}
</style>
