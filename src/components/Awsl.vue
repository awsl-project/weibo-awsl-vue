<template>
  <el-container>
    <el-header>
      <el-select
        v-model="uid"
        @change="chaneUid()"
        placeholder="请选择来源"
        clearable
      >
        <el-option
          :key="producer.uid"
          v-for="producer in producers"
          :label="producer.name"
          :value="producer.uid"
        ></el-option> </el-select
    ></el-header>
    <el-main>
      <div v-if="pic_infos.length">
        <water-fall gap="10px" width="320px" :data="pic_infos" :delay="true">
          <template #default="item">
            <el-card :body-style="{ padding: '2px' }" shadow="hover">
              <img
                style="width: 100%"
                :src="item.url"
                @click="
                  () => {
                    visiableImg = true;
                    srcList = item.srcList;
                  }
                "
              />
              <div style="padding: 14px">
                <el-link :href="item.wb_url" target="_blank">查看原博</el-link>
              </div>
            </el-card>
          </template>
        </water-fall>
        <br />
        <br />
        <div v-if="loading == false">滑动加载更多</div>
        <a-back-top />
      </div>
      <div v-else>
        <el-empty description="暂无数据"></el-empty>
      </div>
      <el-image-viewer
        v-if="visiableImg"
        hide-on-click-modal
        @close="
          () => {
            visiableImg = false;
          }
        "
        :url-list="srcList"
    /></el-main>
  </el-container>

  <el-backtop />
</template>

<script>
import axios from "axios";
import WaterFall from "kuan-vue-waterfall";

export default {
  components: {
    WaterFall,
  },
  data() {
    return {
      uid: "",
      producers: [],
      pic_infos: [],
      total: 0,
      loading: false,
      visiableImg: false,
      srcList: [],
    };
  },
  mounted() {
    this.fetchInit();
    window.addEventListener("scroll", () => {
      const scrollY =
        document.documentElement.scrollTop || document.body.scrollTop; // 滚动条在Y轴上的滚动距离

      const vh =
        document.compatMode === "CSS1Compat"
          ? document.documentElement.clientHeight
          : document.body.clientHeight; // 页面的可视高度（能看见的）

      const allHeight = Math.max(
        document.body.scrollHeight,
        document.documentElement.scrollHeight
      ); // 页面的总高度（所有的）

      if (scrollY + vh >= allHeight) {
        // 当滚动条滑到页面底部
        this.loadMore();
      }
    });
  },
  methods: {
    async fetchData(limit, offset) {
      let url = "/v2/list?limit=" + limit + "&offset=" + offset;
      if (this.uid) {
        url = url + "&uid=" + this.uid;
      }
      let res = await axios.get(url);
      let pic_infos = res.data.map((item) => {
        let pic_info = item.pic_info;
        return {
          key: item.pic_id,
          url: pic_info.large.url,
          srcList: [pic_info.original.url],
          wb_url: item.wb_url,
        };
      });
      this.pic_infos = [...this.pic_infos, ...pic_infos];
    },
    async fetchInit() {
      let producers = await axios.get("/producers");
      this.producers = producers.data;
      if (!this.producers) {
        this.$message({
          showClose: true,
          message: "找不到 Awsl Producers",
          type: "error",
        });
      }
      this.uid = "";
      let res = await axios.get("/v2/list_count");
      this.total = res.data;
      this.fetchData(40, 0);
    },
    async chaneUid() {
      let url = "/v2/list_count";
      if (this.uid) {
        url = url + "?uid=" + this.uid;
      }
      let res = await axios.get(url);
      this.total = res.data;
      this.pic_infos = [];
      this.fetchData(40, 0);
    },
    async loadMore() {
      if (this.loading || this.total <= this.pic_infos.length) return;
      this.loading = true;
      await this.fetchData(5, this.pic_infos.length);
      this.loading = false;
    },
  },
};
</script>
