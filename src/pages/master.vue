<template>
  <div class="main">
    <div class="row">
      <div class="col-12">
        <div id="container"></div>
        <div id="P"><PPip PP="ppip" :node="data" /></div>
      </div>
    </div>
  </div>
</template>

<script>
import { reactive, onMounted, ref, watch } from "vue";
import PPip from "../components/ping.vue";
import { openURL } from "quasar";
import { api } from "boot/axios";
import LogicFlow from "@logicflow/core";
import { Control } from "@logicflow/extension";
import "@logicflow/extension/lib/style/index.css";
import "@logicflow/core/dist/style/index.css";

export default {
  components: { PPip },
  setup() {
    const data = reactive({
      nodes: [],
      edges: [],
    });
    const href = ref("");
    let lf = reactive(Object);

    // function getdata() {
    //   api
    //     .get("data.json")
    //     .then((response) => {
    //       data.nodes = response["data"]["nodes"];
    //       data.edges = response["data"]["edges"];
    //       console.log(data.nodes);
    //       init();
    //     })
    //     .catch((error) => {
    //       console.log(error);
    //     });
    // }
    function init() {
      LogicFlow.use(Control);
      lf = new LogicFlow({
        container: document.querySelector("#container"),
        isSilentMode: true,
        snapline: true,
        stopScrollGraph: true,
        stopZoomGraph: true,
      });
      lf.render(data);

      //添加事件监听，点击跳转
      lf.on("element:click", ({ data, e }) => {
        console.log(data.properties.href);
        // href.value = data.properties.href;
        openURL(data.properties.href);
      });
    }
    function UrlPing() {
      api
        .get("")
        .then((response) => {
          console.log(response);
        })
        .catch((error) => {
          console.log(error);
        });
    }
    //使用LocalStorage保存数据进行可持续化 替代读取本地文件
    function LoadLs() {
      if (window.localStorage) {
      } else {
        alert("请使用最新版本浏览器");
      }
      var cons = localStorage.getItem("data");
      if (cons == null) {
        alert("请先到添加页面保存布局");
      } else {
        var Jcons = JSON.parse(cons);
        data.nodes = Jcons["nodes"];
        data.edges = Jcons["edges"];
      }

      init();
    }
    onMounted(() => {
      LoadLs();
    });

    return { data, init, href, UrlPing, LoadLs };
  },
};
</script>
<style>
#container {
  height: 600px;
}
</style>
