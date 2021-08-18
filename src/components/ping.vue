<template>
  <component :is="PP">
    <q-btn @click="d2List()">测试速度</q-btn>
    <div class="row">
      <div class="col-4">
        <div>hosts</div>
        <ul>
          <li v-for="(item, index) in Hosts" :key="index">{{ item }}</li>
        </ul>
      </div>
      <div class="col-4">
        <div>url</div>
        <ul>
          <li v-for="(item, index) in IpList" :key="index">{{ item }}</li>
        </ul>
      </div>
      <div class="col-4">
        <div>ping</div>
        <ul>
          <li v-for="(item, index) in Ms" :key="index">{{ item }}</li>
        </ul>
      </div>
    </div>
  </component>
</template>
<script>
import Ping from "ping.js";
import { reactive, onMounted, ref, watch } from "vue";

export default {
  name: "PPip",
  props: {
    PP: {
      type: String,
      default: "div",
    },
    node: {
      type: Object,
      default: "",
    },
  },
  setup(props) {
    const data = reactive({
      data: {},
    });
    const HrefLength = ref("");
    const Hosts = ref([]);
    const IpList = ref([]);
    const Ms = ref([]);
    function d2List() {
      data.data = props.node;
      Hosts.value = [];
      IpList.value = [];
      Ms.value = [];
      HrefLength.value = data.data.nodes.length;
      for (var i = 0; i < HrefLength.value; i++) {
        Hosts.value.push(data.data.nodes[i].text.value);
        IpList.value.push(data.data.nodes[i].properties.href);
      }
      PP();
    }

    function PP() {
      var p = new Ping();
      for (var i = 0; i < HrefLength.value; i++) {
        p.ping(IpList.value[i])
          .then((data) => {
            console.log(data);
            Ms.value.push(data);
          })
          .catch((data) => {
            console.log(data);
            Ms.value.push(data);
          });
      }
      console.log(Ms.value, "Ms");
    }

    onMounted(() => {});

    return { d2List, data, Hosts, IpList, Ms };
  },
};
</script>
