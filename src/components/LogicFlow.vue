<template>
  <component :is="lf">
    <div id="main">
      <div height="30">
        <q-toggle v-model="ndata.mapflag" label="MiniMap"> </q-toggle>
        <q-btn color="primary" rounded @click="SaveLs()">保存修改 </q-btn>
      </div>

      <div class="row" style="height: 100%">
        <div class="col-9">
          <div id="container"></div>
        </div>
        <div class="col-3">
          <div class="InputPanel" v-if="ndata.InputPanelFlag">
            <h4>{{ ndata.title }}</h4>
            <q-form @submit="Show_set()" @reset="onReset">
              <q-input label="节点ID" v-model="ndata.Input.newnode.id" />
              <q-input
                label="节点描述(若不显示多点几次修改)"
                v-model="ndata.Input.newnode.text.value"
              />
              <div>
                <q-radio
                  v-model="ndata.Input.newnode.type"
                  val="circle"
                  label="圆形"
                />
                <q-radio
                  v-model="ndata.Input.newnode.type"
                  val="rect"
                  label="方形"
                />
                <q-radio
                  v-model="ndata.Input.newnode.type"
                  val="diamond"
                  label="菱形"
                />
              </div>
              <q-input label="x坐标" disable v-model="ndata.Input.newnode.x" />
              <q-input label="y坐标" disable v-model="ndata.Input.newnode.y" />
              <q-input
                label="跳转链接(http://xxx)"
                v-model="ndata.Input.newnode.properties.href"
              />

              <div>
                <q-btn
                  label="添加新节点"
                  color="primary"
                  @click="Show_add()"
                  class="q-ml-sm"
                />
                <q-btn label="修改" flat color="primary" @click="Show_set()" />
              </div>
            </q-form>
          </div>
        </div>
      </div>
    </div>
  </component>
</template>

<script>
import { reactive, onMounted, ref, watch } from "vue";
import LogicFlow from "@logicflow/core";
import { api } from "boot/axios";

import {
  DndPanel,
  SelectionSelect,
  Menu,
  Control,
  Snapshot,
  MiniMap,
} from "@logicflow/extension";
import "@logicflow/extension/lib/style/index.css";
import "@logicflow/core/dist/style/index.css";

export default {
  name: "LogicFlow",
  props: {
    lf: {
      type: String,
      default: "div",
    },
  },

  setup() {
    const data = reactive({
      // 节点
      nodes: [
        {
          id: 1,
          type: "circle",
          x: 100,
          y: 150,
          text: "你好",
        },
      ],
      // 边
      edges: [],
      shapeList1: [
        {
          text: "选区",
          icon: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAYAAAH6ji2bAAAABGdBTUEAALGPC/xhBQAAAOVJREFUOBGtVMENwzAIjKP++2026ETdpv10iy7WFbqFyyW6GBywLCv5gI+Dw2Bluj1znuSjhb99Gkn6QILDY2imo60p8nsnc9bEo3+QJ+AKHfMdZHnl78wyTnyHZD53Zzx73MRSgYvnqgCUHj6gwdck7Zsp1VOrz0Uz8NbKunzAW+Gu4fYW28bUYutYlzSa7B84Fh7d1kjLwhcSdYAYrdkMQVpsBr5XgDGuXwQfQr0y9zwLda+DUYXLaGKdd2ZTtvbolaO87pdo24hP7ov16N0zArH1ur3iwJpXxm+v7oAJNR4JEP8DoAuSFEkYH7cAAAAASUVORK5CYII=",
          callback: () => {
            lf.updateEditConfig({
              stopMoveGraph: true,
            });
          },
        },
        {
          type: "circle",
          text: "开始",
          icon: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAYAAAH6ji2bAAAABGdBTUEAALGPC/xhBQAAAnBJREFUOBGdVL1rU1EcPfdGBddmaZLiEhdx1MHZQXApraCzQ7GKLgoRBxMfcRELuihWKcXFRcEWF8HBf0DdDCKYRZpnl7p0svLe9Zzbd29eQhTbC8nv+9zf130AT63jvooOGS8Vf9Nt5zxba7sXQwODfkWpkbjTQfCGUd9gIp3uuPP8bZ946g56dYQvnBg+b1HB8VIQmMFrazKcKSvFW2dQTxJnJdQ77urmXWOMBCmXM2Rke4S7UAW+/8ywwFoewmBps2tu7mbTdp8VMOkIRAkKfrVawalJTtIliclFbaOBqa0M2xImHeVIfd/nKAfVq/LGnPss5Kh00VEdSzfwnBXPUpmykNss4lUI9C1ga+8PNrBD5YeqRY2Zz8PhjooIbfJXjowvQJBqkmEkVnktWhwu2SM7SMx7Cj0N9IC0oQXRo8xwAGzQms+xrB/nNSUWVveI48ayrFGyC2+E2C+aWrZHXvOuz+CiV6iycWe1Rd1Q6+QUG07nb5SbPrL4426d+9E1axKjY3AoRrlEeSQo2Eu0T6BWAAr6COhTcWjRaYfKG5csnvytvUr/WY4rrPMB53Uo7jZRjXaG6/CFfNMaXEu75nG47X+oepU7PKJvvzGDY1YLSKHJrK7vFUwXKkaxwhCW3u+sDFMVrIju54RYYbFKpALZAo7sB6wcKyyrd+aBMryMT2gPyD6GsQoRFkGHr14TthZni9ck0z+Pnmee460mHXbRAypKNy3nuMdrWgVKj8YVV8E7PSzp1BZ9SJnJAsXdryw/h5ctboUVi4AFiCd+lQaYMw5z3LGTBKjLQOeUF35k89f58Vv/tGh+l+PE/wG0rgfIUbZK5AAAAABJRU5ErkJggg==",
        },
        {
          type: "rect",
          text: "用户任务",
          icon: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABMAAAATCAYAAAEFVwZaAAAABGdBTUEAALGPC/xhBQAAAqlJREFUOBF9VM9rE0EUfrMJNUKLihGbpLGtaCOIR8VjQMGDePCgCCIiCNqzCAp2MyYUCXhUtF5E0D+g1t48qAd7CCLqQUQKEWkStcEfVGlLdp/fm3aW2QQdyLzf33zz5m2IsAZ9XhDpyaaIZkTS4ASzK41TFao88GuJ3hsr2pAbipHxuSYyKRugagICGANkfFnNh3HeE2N0b3nN2cgnpcictw5veJIzxmDamSlxxQZicq/mflxhbaH8BLRbuRwNtZp0JAhoplVRUdzmCe/vO27wFuuA3S5qXruGdboy5/PRGFsbFGKo/haRtQHIrM83bVeTrOgNhZReWaYGnE4aUQgTJNvijJFF4jQ8BxJE5xfKatZWmZcTQ+BVgh7s8SgPlCkcec4mGTmieTP4xd7PcpIEg1TX6gdeLW8rTVMVLVvb7ctXoH0Cydl2QOPJBG21STE5OsnbweVYzAnD3A7PVILuY0yiiyDwSm2g441r6rMSgp6iK42yqroI2QoXeJVeA+YeZSa47gZdXaZWQKTrG93rukk/l2Al6Kzh5AZEl7dDQy+JjgFahQjRopSxPbrbvK7GRe9ePWBo1wcU7sYrFZtavXALwGw/7Dnc50urrHJuTPSoO2IMV3gUQGNg87IbSOIY9BpiT9HV7FCZ94nPXb3MSnwHn/FFFE1vG6DTby+r31KAkUktB3Qf6ikUPWxW1BkXSPQeMHHiW0+HAd2GelJsZz1OJegCxqzl+CLVHa/IibuHeJ1HAKzhuDR+ymNaRFM+4jU6UWKXorRmbyqkq/D76FffevwdCp+jN3UAN/C9JRVTDuOxC/oh+EdMnqIOrlYteKSfadVRGLJFJPSB/ti/6K8f0CNymg/iH2gO/f0DwE0yjAFO6l8JaR5j0VPwPwfaYHqOqrCI319WzwhwzNW/aQAAAABJRU5ErkJggg==",
          className: "important-node",
        },
        {
          type: "rect",
          text: "系统任务",
          icon: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABMAAAATCAYAAAEFVwZaAAAABGdBTUEAALGPC/xhBQAAAqlJREFUOBF9VM9rE0EUfrMJNUKLihGbpLGtaCOIR8VjQMGDePCgCCIiCNqzCAp2MyYUCXhUtF5E0D+g1t48qAd7CCLqQUQKEWkStcEfVGlLdp/fm3aW2QQdyLzf33zz5m2IsAZ9XhDpyaaIZkTS4ASzK41TFao88GuJ3hsr2pAbipHxuSYyKRugagICGANkfFnNh3HeE2N0b3nN2cgnpcictw5veJIzxmDamSlxxQZicq/mflxhbaH8BLRbuRwNtZp0JAhoplVRUdzmCe/vO27wFuuA3S5qXruGdboy5/PRGFsbFGKo/haRtQHIrM83bVeTrOgNhZReWaYGnE4aUQgTJNvijJFF4jQ8BxJE5xfKatZWmZcTQ+BVgh7s8SgPlCkcec4mGTmieTP4xd7PcpIEg1TX6gdeLW8rTVMVLVvb7ctXoH0Cydl2QOPJBG21STE5OsnbweVYzAnD3A7PVILuY0yiiyDwSm2g441r6rMSgp6iK42yqroI2QoXeJVeA+YeZSa47gZdXaZWQKTrG93rukk/l2Al6Kzh5AZEl7dDQy+JjgFahQjRopSxPbrbvK7GRe9ePWBo1wcU7sYrFZtavXALwGw/7Dnc50urrHJuTPSoO2IMV3gUQGNg87IbSOIY9BpiT9HV7FCZ94nPXb3MSnwHn/FFFE1vG6DTby+r31KAkUktB3Qf6ikUPWxW1BkXSPQeMHHiW0+HAd2GelJsZz1OJegCxqzl+CLVHa/IibuHeJ1HAKzhuDR+ymNaRFM+4jU6UWKXorRmbyqkq/D76FffevwdCp+jN3UAN/C9JRVTDuOxC/oh+EdMnqIOrlYteKSfadVRGLJFJPSB/ti/6K8f0CNymg/iH2gO/f0DwE0yjAFO6l8JaR5j0VPwPwfaYHqOqrCI319WzwhwzNW/aQAAAABJRU5ErkJggg==",
          cls: "import_icon",
        },
        {
          type: "diamond",
          text: "条件判断",
          icon: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABUAAAAVCAYAAAHeEJUAAAAABGdBTUEAALGPC/xhBQAAAvVJREFUOBGNVEFrE0EU/mY3bQoiFlOkaUJrQUQoWMGePLX24EH0IIoHKQiCV0G8iE1covgLiqA/QTzVm1JPogc9tIJYFaQtlhQxqYjSpunu+L7JvmUTU3AgmTfvffPNN++9WSA1DO182f6xwILzD5btfAoQmwL5KJEwiQyVbSVZ0IgRyV6PTpIJ81E5ZvqfHQR0HUOBHW4L5Et2kQ6Zf7iAOhTFAA8s0pEP7AXO1uAA52SbqGk6h/6J45LaLhO64ByfcUzM39V7ZiAdS2yCePPEIQYvTUHqM/n7dgQNfBKWPjpF4ISk8q3J4nB11qw6X8l+FsF3EhlkEMfrjIer3wJTLwS2aCNcj4DbGxXTw00JmAuO+Ni6bBxVUCvS5d9aa04+so4pHW5jLTywuXAL7jJ+D06sl82Sgl2JuVBQn498zkc2bGKxULHjCnSMadBKYDYYHAtsby1EQ5lNGrQd4Y3v4Zo0XdGEmDno46yCM9Tk+RiJmUYHS/aXHPNTcjxcbTFna000PFJHIVZ5lFRqRpJWk9/+QtlOUYJj9HG5pVFEU7zqIYDVsw2s+AJaD8wTd2umgSCCyUxgGsS1Y6TBwXQQTFuZaHcd8gAGioE90hlsY+wMcs30RduYtxanjMGal8H5dMW67dmT1JFtYUEe8LiQLRsPZ6IIc7A4J5tqco3T0pnv/4u0kyzrYUq7gASuEyI8VXKvB9Odytv6jS/PNaZBln0nioJG/AVQRZvApOdhjj3Jt8QC8Im09SafwdBdvIpztpxWxpeKCC+EsFdS8DCyuCn2munFpL7ctHKp+Xc5cMybeIyMAN33SPL3ZR9QV1XVwLyzHm6Iv0/yeUuUb7PPlZC4D4HZkeu6dpF4v9j9MreGtMbxMMRLIcjJic9yHi7WQ3yVKzZVWUr5UrViJvn1FfUlwe/KYVfYyWRLSGNu16hR01U9IacajXPei0wx/5BqgInvJN+MMNtNme7ReU9SBbgntovn0kKHpFg7UogZvaZiOue/q1SBo9ktHzQAAAAASUVORK5CYII=",
        },
        {
          type: "circle",
          text: "结束",
          icon: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAYAAAH6ji2bAAAABGdBTUEAALGPC/xhBQAAA1BJREFUOBFtVE1IVUEYPXOf+tq40Y3vPcmFIdSjIorWoRG0ERWUgnb5FwVhYQSl72oUoZAboxKNFtWiwKRN0M+jpfSzqJAQclHo001tKkjl3emc8V69igP3znzfnO/M9zcDcKT67azmjYWTwl9Vn7Vumeqzj1DVb6cleQY4oAVnIOPb+mKAGxQmKI5CWNJ2aLPatxWa3aB9K7/fB+/Z0jUF6TmMlFLQqrkECWQzOZxYGjTlOl8eeKaIY5yHnFn486xBustDjWT6dG7pmjHOJd+33t0iitTPkK6tEvjxq4h2MozQ6WFSX/LkDUGfFwfhEZj1Auz/U4pyAi5Sznd7uKzznXeVHlI/Aywmk6j7fsUsEuCGADrWARXXwjxWQsUbIupDHJI7kF5dRktg0eN81IbiZXiTESic50iwS+t1oJgL83jAiBupLDCQqwziaWSoAFSeIR3P5Xv5az00wyIn35QRYTwdSYbz8pH8fxUUAtxnFvYmEmgI0wYXUXcCCSpeEVpXlsRhBnCEATxWylL9+EKCAYhe1NGstUa6356kS9NVvt3DU2fd+Wtbm/+lSbylJqsqkSm9CRhvoJVlvKPvF1RKY/FcPn5j4UfIMLn8D4UYb54BNsilTDXKnF4CfTobA0FpoW/LSp306wkXM+XaOJhZaFkcNM82ASNAWMrhrUbRfmyeI1FvRBTpN06WKxa9BK0o2E4Pd3zfBBEwPsv9sQBnmLVbLEIZ/Xe9LYwJu/Er17W6HYVBc7vmuk0xUQ+pqxdom5Fnp55SiytXLPYoMXNM4u4SNSCFWnrVIzKG3EGyMXo6n/BQOe+bX3FClY4PwydVhthOZ9NnS+ntiLh0fxtlUJHAuGaFoVmttpVMeum0p3WEXbcll94l1wM/gZ0Ccczop77VvN2I7TlsZCsuXf1WHvWEhjO8DPtyOVg2/mvK9QqboEth+7pD6NUQC1HN/TwvydGBARi9MZSzLE4b8Ru3XhX2PBxf8E1er2A6516o0w4sIA+lwURhAON82Kwe2iDAC1Watq4XHaGQ7skLcFOtI5lDxuM2gZe6WFIotPAhbaeYlU4to5cuarF1QrcZ/lwrLaCJl66JBocYZnrNlvm2+MBCTmUymPrYZVbjdlr/BxlMjmNmNI3SAAAAAElFTkSuQmCC",
        },
        {
          text: "导出图片",
          icon: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABMAAAATCAYAAAByUDbMAAAACXBIWXMAABJNAAASTQHzl8SnAAACDklEQVQ4T62Uz2sTQRTH33d2qdjcKtST9aSHCDWhUj1YUKF3DxUVb/ZQ8KJITUAEuxWMP9qjUgrtxYOgF/8AQSyeRG1SpQVbUPAk+APUhGKS9/zOoRJ2N7EBB3Z35r15n/nOvpkHeWRB7oNeg8iEQfpE2Es0fVUuhEfj5sG7lmlUpb46hd/eF+Y+6jQgBRO5ZyYbIuzFmwXrrSYPCVQvi+mk2ymfhq7b8OsItZA6zntQuRBcijOSY0PuTnMcphF3sYv+Hgo5oBmdZn/SEbSbalaTgUnLwZuyD4I5MVsStdN+hgme8XUxf9sOOSpL+0lJEi2Vq3j/s+r6lovhWYON0PT9VxUnKeitwOZcalQH40aEH9lbNgBxE1Qx78dO3DmqfRx2iEt1ZaesZweaDwT4RpUlP+lNEWv8rHUFY9Z6tdceimFY1Y16Va0rbhuWK9WPqdP7DB5QuFOVIl5sgXhU+gNrjrSF5WdsrzYae+DcEMzOGHCYmXupwJGVK3jXqsipjHHbs6mwwVL9hKk+dc4nW2p8npvZWKXmnkgEbQX5vokykUAqbGUzXMpn5LiqfKlvyvrWdYlD4uNUGFdvLFNNfPK/xl2fs07A/wvjVUgpE53Wb+cz8en6zExk203Zjp2Q/eR8DVnDFlhGiiwtvF6+nnXXPIgRF7i/CH8rLTBuYv1dFBEyeIy5M4pYLNfcjT92x8ZPzuMhQgAAAABJRU5ErkJggg==",
          callback: () => {
            // eslint-disable-next-line @typescript-eslint/no-unsafe-call
            lf.getSnapshot();
          },
        },
      ],
      clickNodeId: "",
      clickEdgeId: "",
    });
    const ndata = reactive({
      title: "信息",
      addflag: true,
      InputPanelFlag: true, //添加节点的面板，false为有需要再显示
      Input: {
        typeflag: 0, //节点信息，用作修改的判断，0为节点，1为边
        //新节点/边的信息
        newnode: {
          id: String,
          text: {
            value: "",
            x: Number,
            y: Number,
          },
          type: "",
          x: Number,
          y: Number,
          properties: {
            href: "",
          },
        },
        newedge: {
          type: "polyline",
          sourceNodeId: "",
          targetNodeId: "",
          text: "",
        },
      },
      mapflag: false, //0关闭
    });

    let lf = reactive(Object);

    function LoadLs() {
      var cons = localStorage.getItem("data");
      if (cons == null) {
        draw();
      } else {
        var Jcons = JSON.parse(cons);
        data.nodes = Jcons["nodes"];
        data.edges = Jcons["edges"];
        draw();
      }
    }
    function draw() {
      // LF原生组件框
      LogicFlow.use(DndPanel);
      // 框选插件
      LogicFlow.use(SelectionSelect);
      // 右键菜单
      LogicFlow.use(Menu);
      // 控制面板 启用后自动支持"ctrl+z|y"
      LogicFlow.use(Control);
      // 导出图片
      LogicFlow.use(Snapshot);
      // miniMap
      LogicFlow.use(MiniMap);
      lf = new LogicFlow({
        container: document.querySelector("#container"),
        stopScrollGraph: true,
        stopZoomGraph: true,
        grid: {
          type: "dot",
          size: 20,
        },
        keyboard: {
          enabled: true,
        },
      });

      //右键菜单追加选项
      lf.addMenuConfig({
        nodeMenu: [
          {
            text: "向右追加节点",
            callback(node) {
              //node为被点击节点数据
              //重写Showadd方法
              ShowInputpanel();
              let newid = createHash();
              console.log(newid);
              ndata.Input.newnode.id = newid;
              ndata.Input.newnode.x = node.x + 150;
              ndata.Input.newnode.y = node.y;
              ndata.Input.newedge.sourceNodeId = node.id;
              ndata.Input.newedge.targetNodeId = newid;
            },
          },
          {
            text: "向下追加节点",
            callback(node) {
              //node为被点击节点数据
              //重写Showadd方法
              ShowInputpanel();
              let newid = createHash();
              console.log(newid);
              ndata.Input.newnode.id = newid;
              ndata.Input.newnode.x = node.x;
              ndata.Input.newnode.y = node.y + 150;
              ndata.Input.newedge.sourceNodeId = node.id;
              ndata.Input.newedge.targetNodeId = newid;
            },
          },
          {
            text: "向左追加节点",
            callback(node) {
              //node为被点击节点数据
              //重写Showadd方法
              ShowInputpanel();
              let newid = createHash();
              console.log(newid);
              ndata.Input.newnode.id = newid;
              ndata.Input.newnode.x = node.x - 150;
              ndata.Input.newnode.y = node.y;
              ndata.Input.newedge.sourceNodeId = node.id;
              ndata.Input.newedge.targetNodeId = newid;
            },
          },
          {
            text: "向上追加节点",
            callback(node) {
              //node为被点击节点数据
              //重写Showadd方法
              ShowInputpanel();
              let newid = createHash();
              console.log(newid);
              ndata.Input.newnode.id = newid;
              ndata.Input.newnode.x = node.x;
              ndata.Input.newnode.y = node.y - 150;
              ndata.Input.newedge.sourceNodeId = node.id;
              ndata.Input.newedge.targetNodeId = newid;
            },
          },
          //节点坐标原点在左上角，向下y增加，向上y减小。
        ],
      });

      // 原生拖拽框的数据渲染，决定拖拽框中的组件
      lf.setPatternItems(data.shapeList1);
      lf.render(data);
      // MiniMap.show(1050,300);

      // id为1的图形居中
      // lf.focusOn({
      //   id: 1
      // });

      // 获取数据
      // let info = lf.getGraphData()

      // 点击事件监听  添加连线type判断点击的是节点或连线
      lf.on("element:click", ({ data, e }) => {
        // console.log(lf.getGraphData(),'1')
        if (data.type == "polyline") {
          ndata.typeflag = 1;
          ndata.title = "边信息";
          ndata.Input.newnode.id = data.id;
          ndata.Input.newnode.text.value = data.text.value;
          console.log(data.text.value);
        } else {
          ndata.typeflag = 0;
          ndata.title = "节点信息";
          // 点击节点，输入框中显示节点信息，通过修改输入框中节点信息，实现直接修改节点
          ndata.Input.newnode.id = data.id;
          ndata.Input.newnode.x = data.x;
          ndata.Input.newnode.y = data.y;
          ndata.Input.newnode.type = data.type;
          ndata.Input.newnode.properties.href = data.properties.href;
          console.log(ndata.mapflag);
          // 拖拽节点text.value值为空的判断
          if (typeof data.text == "undefined") {
            ndata.Input.newnode.text.value = "undefined";
            console.log(data);
          } else {
            ndata.Input.newnode.text.x = ndata.Input.newnode.x;
            ndata.Input.newnode.text.y = ndata.Input.newnode.y;
            ndata.Input.newnode.text.value = data.text.value;
          }
        }
      });
    }

    //id使用24位哈希
    function createHash(hashLength) {
      // 默认长度 24
      return Array.from(Array(Number(hashLength) || 24), () =>
        Math.floor(Math.random() * 36).toString(36)
      ).join("");
    }

    //自定义节点按钮开始
    function ShowInputpanel() {
      ndata.InputPanelFlag = true;
    }

    //添加新节点
    function Show_add() {
      lf.addNode(ndata.Input.newnode);
      lf.createEdge(ndata.Input.newedge);
      console.log(lf.getGraphData());
      ndata.InputPanelFlag = false;
    }

    //修改节点
    function Show_set() {
      if (ndata.Input.typeflag == 0) {
        lf.setNodeData(ndata.Input.newnode);
      } else {
        lf.setEdgeDtat(ndata.Input.newedge);
      }
    }

    function Show_close() {
      ndata.InputPanelFlag = false;
    }
    //自定义节点按钮结束

    //导出图片
    function download(filename, text) {
      var element = document.createElement("a");
      element.setAttribute(
        "href",
        "data:text/plain;charset=utf-8," + encodeURIComponent(text)
      );
      element.setAttribute("download", filename);
      element.style.display = "none";
      document.body.appendChild(element);
      element.click();
      document.body.removeChild(element);
    }

    function handleDoenload() {
      lf.getSnapshot();
    }
    //导出json数据
    // function exportJson() {
    //   const adapterData = lf.getGraphData();
    //   download("logic-flow.json", JSON.stringify(adapterData));
    //   console.log(adapterData);
    // }

    //保存数据到LocalStorage 使用LocalStorage保存数据进行可持续化 替代读取本地文件
    function SaveLs() {
      const adapterData = lf.getGraphData();
      localStorage.setItem("data", JSON.stringify(adapterData));
    }

    // minimap 开关
    watch(ndata, (flag) => {
      if (flag.mapflag == true) {
        MiniMap.show(1050, 300);
        console.log(flag.mapflag);
      } else {
        MiniMap.hide();
        console.log(flag.mapflag);
      }
    });

    onMounted(() => {
      LoadLs();
    });

    return {
      data,
      ndata,
      ShowInputpanel,
      Show_add,
      Show_set,
      Show_close,
      handleDoenload,
      SaveLs,
      LoadLs,
    };
  },
};
</script>

<style>
#container {
  height: 600px;
}
.InputPanel {
  z-index: 999;
  width: 100%;
  height: 100%;
  background-color: #ffffff;
  border: 2px solid #888888;
  box-shadow: 10px 10px 5px #888888;
}
</style>
