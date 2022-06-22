<template>
  <a-spin tip="配置加载中..." :spinning="spinning1 || spinning2">
    <div id="parent-div">
      <a-row>
        <a-form layout="inline" :model="config">
          <a-form-item label="● 控制台地址">
            <a-input v-model:value="config.server"></a-input>
          </a-form-item>
          <a-form-item>
            <a-button @click="config.addItemModalShow = true" type="primary">新建连接</a-button>
          </a-form-item>
        </a-form>
      </a-row>
      <br>
      <a-row :gutter="1">
        <a-col :lg="24" :xl="12" v-for="(item) in items" :key="item.name">
          <a-card size="small" :title="item.name">
            <template #extra>
              <a-space>
                <label v-if="item.type == 'TC'">{{ "服务器[" + item.comServer + "]: " + (item.comStatus == 0 ? "未连接"
                    : "已连接")
                }}</label>
                <label v-if="item.type == 'TS'">{{ "端口[" + item.comServer + "]: " + (item.comStatus == 0 ? "未监听"
                    : "已监听")
                }}</label>
                <a-button size="small" v-if="item.type == 'TC' && item.comStatus != 1" @click="connect(item.name)">连接
                </a-button>
                <a-button size="small" v-if="item.type == 'TC' && item.comStatus == 1" @click="disconnect(item.name)">断开
                </a-button>
                <a-button size="small" @click="delItem(item.name)" danger>删除</a-button>
              </a-space>
            </template>
            <a-row>
              <a-col :span="12">
                <a-space><label>发送字段 </label>
                  <a-button v-if="item.type.endsWith('C') && item.sendType == 'SD'"
                    :disabled="item.type == 'TC' && (item.comStatus == 0 || item.send.length == 0)" @click="send(item.name)">发送</a-button>
                </a-space>
                <ParamsList :type="'send'" :params="item.send" :name="item.name" :addParamModalOk="addParamModalOk"
                  :changeValue="changeValue">
                </ParamsList>
              </a-col>
              <a-col :span="12">
                <a-space><label>接收字段 </label>

                </a-space>
                <ParamsList :type="'recv'" :params="item.recv" :name="item.name" :addParamModalOk="addParamModalOk">
                </ParamsList>
              </a-col>
            </a-row>
          </a-card>
        </a-col>
      </a-row>
      <!-- 新建item模态框 -->
      <a-modal v-model:visible="config.addItemModalShow" title="创建连接" @ok="addItemModalOk" width="700px" ok-text="创建"
        cancel-text="取消">
        <a-form :model="config">
          <a-form-item label="● 连接类型">
            <a-radio-group v-model:value="config.itemType" button-style="solid">
              <a-radio-button value="TC">TCP客户端</a-radio-button>
              <a-radio-button value="TS">TCP服务器</a-radio-button>
              <a-radio-button value="UC">UDP客户端</a-radio-button>
              <a-radio-button value="US">UDP服务端</a-radio-button>
            </a-radio-group>
          </a-form-item>
          <a-form-item label="● 发送类型">
            <a-radio-group v-model:value="config.sendType" button-style="solid">
              <a-radio-button value="SD">手动</a-radio-button>
              <a-radio-button value="JG">间隔</a-radio-button>
              <a-radio-button value="CF">触发</a-radio-button>
            </a-radio-group>
          </a-form-item>
          <a-form-item label="● 间隔时间/ms" v-if="config.sendType == 'JG'">
            <a-input :allowClear="true" v-model:value="config.sendInterval" type="number" style="width:200px"></a-input>
          </a-form-item>
          <a-form-item label="● 接收类型">
            <a-radio-group v-model:value="config.recvType" button-style="solid">
              <a-radio-button value="DC">定长</a-radio-button>
              <a-radio-button value="JSF">结束符</a-radio-button>
            </a-radio-group>
          </a-form-item>
          <a-form-item v-if="config.recvType == 'JSF'" label="● 结束符(16进制数字表示)">
            <a-input :addon-before="'0x'" :allowClear="true" v-model:value="config.recvJsf" type="number"
              style="width:150px"></a-input>
          </a-form-item>
        </a-form>
        <a-form layout="inline" :model="config">
          <a-form-item v-if="config.itemType == 'TC' || config.itemType == 'UC'" label="● 服务器地址:端口">
            <a-input :allowClear="true" v-model:value="config.comServer"></a-input>
          </a-form-item>
          <br><br>
          <a-form-item label="● 项目名称">
            <a-input style="width:300px" :allowClear="true" :addon-before="getItemNamePrev()"
              v-model:value="config.newItemName" @keyup.enter="createItem">
            </a-input>
          </a-form-item>
        </a-form>
      </a-modal>
      <!-- 添加字段模态框 -->
      <a-modal v-model:visible="config.addParamModalShow" width="500px"
        :title="`${tmp.itemName}: 添加${tmp.paramTye == 'send' ? '发送' : '接收'}字段`" ok-text="确认" cancel-text="取消"
        @ok="addParamModalOk"></a-modal>
    </div>
  </a-spin>
</template>

<script>
import ParamsList from "@/components/ParamsList.vue"
import { ExclamationCircleOutlined } from '@ant-design/icons-vue';
import { createVNode } from 'vue';
export default {
  name: 'App',
  data() {
    return {
      spinning1: true,
      spinning2: true,
      itemWs: null,
      config: {
        server: "127.0.0.1:20003",
        itemType: "TC",
        newItemName: "",
        addParamModalShow: false,
        addItemModalShow: false,
        comServer: "",
        sendType: "SD",
        recvType: "DC"
      },
      // status 0未连接 1已连接
      items: [
        /*{
          con: null, type: "TC", name: "测试item", createTime: new Date(), wsStatus: 0, comStatus: 0,
          recv: [{ name: "param1", type: "int32", value: 4, length: 5 }, { name: "param2", type: "str", value: "ceshi", length: 5 }],
          send: [{ name: "param2", type: "str", value: "ceshi", length: 5 }, { name: "param1", type: "int32", value: 4, length: 5 }],
        }*/
      ],
      tmp: {}
    };
  },
  components: {
    ParamsList
  },
  mounted() {
    this.getAllItems();
    this.initItemWs();
  },
  methods: {
    getItemNamePrev() {
      return (this.config.itemType == 'TC' ? 'TCP客户端' : this.config.itemType == 'TS' ? 'TCP服务器' : this.config.itemType == 'UC' ? 'UDP客户端' : 'UDP服务端') + '-';
    },
    getItemByName(name) {
      let res = null;
      this.items.forEach(item => {
        if (item.name == name) {
          res = item;
        }
      });
      return res;
    },
    initItemWs() {
      this.itemWs = new WebSocket(`ws://${this.config.server}/item`);
      this.itemWs.onopen = () => {
        this.spinning2 = false;
      };
      this.itemWs.onmessage = (e) => {
        let res = JSON.parse(e.data);
        let action = res.action;
        if (action == "update") {
          this.items.splice(0, 0, res.item);
          this.initComWs(res.item.name);
        } else if (action == "updateParams") {
          let item = this.getItemByName(res.name);
          item[res.type] = res.params;
        }
      }
    },
    initComWs(name) {
      let item = this.getItemByName(name);
      item.client = new WebSocket(`ws://${this.config.server}/connect?name=${item.name}`);
      // item.client.onopen = (e) => {
      //   item.client.send("connect");
      // }
      item.client.onmessage = (e) => {
        console.log("onmessage", e);
        let res = JSON.parse(e.data);
        let { action, updData } = res;
        if (action == "update") {
          let { key, value } = updData;
          let keys = key.split(".");
          let tmp = "item";
          for (let i = 0; i < keys.length; i++) {
            tmp += `['${keys[i]}']`;
          }
          tmp += "=" + value;
          eval(tmp);
        }
      }
      item.client.onclose = (e) => {
        console.log("onclose", e);
      }
    },
    initAllComWs() {
      this.items.forEach(item => {
        this.initComWs(item.name);
      });
    },
    sendItemMsg(type, name, data) {
      let msg = { type: type };
      msg[name] = data;
      this.itemWs.send(JSON.stringify(msg));
    },
    getAllItems() {
      let ws = new WebSocket(`ws://${this.config.server}/getAll`);
      ws.onopen = () => {
        ws.send(1);
      }
      ws.onerror = () => {
        this.$message.error("读取配置失败");
      }
      ws.onmessage = (e) => {
        let items = JSON.parse(e.data);
        this.items = items.reverse();
        this.initAllComWs();
        this.spinning1 = false;
        ws.close();
      }
    },
    createItem(cb) {
      if (this.config.sendType == "JG" && (!this.config.sendInterval || this.config.sendInterval <= 0)) {
        this.$message.info(`输入正确间隔时间`);
        return false;
      }
      if (this.config.itemType.endsWith("C") && !this.config.comServer) {
        this.$message.info(`输入服务端地址`);
        return false;
      }
      if (!this.config.newItemName) {
        this.$message.info(`输入项目名称`);
        return false;
      }
      let hasItem = false;
      let name = this.getItemNamePrev() + this.config.newItemName;
      let type = this.config.itemType;
      let sendType = this.config.sendType;
      let comServer = this.config.comServer;
      let sendInterval = this.config.sendInterval;
      this.items.forEach(item => {
        if (item.name == name) {
          hasItem = true;
        }
      });
      if (hasItem) {
        this.$message.error(`名称“${name}”重复`);
        return false;
      } else {
        let item = { type, name, createTime: new Date().getTime(), recv: [], send: [], comStatus: 0, wsStatus: 0, sendType, comServer, sendInterval };
        this.config.newItemName = "";
        this.sendItemMsg("add", "item", item);
        cb();
      }
    }, connect(name) {
      let item = this.getItemByName(name);
      item.client.send("connect");
    }, send(name) {
      let item = this.getItemByName(name);
      item.client.send("send");
    },
    disconnect(name) {
      let item = this.getItemByName(name);
      item.client.send("disconnect");
    }, delItem(name) {
      let self = this;
      this.$confirm({
        title: '确定删除此项目?',
        icon: createVNode(ExclamationCircleOutlined),
        okText: "确定",
        cancelText: "取消",
        content: createVNode('div', { style: 'color:red;' }, '此操作无法恢复'),
        onOk() {
          self.disconnect(name);
          for (let i = 0; i < self.items.length; i++) {
            if (self.items[i].name == name) {
              self.sendItemMsg("del", "name", name);
              self.items.splice(i, 1);
            }
          }
        },
        onCancel() {
          console.log('Cancel');
        },
      });
    }, addParamModalOk(name, param, type) {
      this.sendItemMsg("addParam", "data", { name, param, type });
    }, addItemModalOk() {
      this.createItem(() => {
        this.config.addItemModalShow = false;
      });
    }, changeValue(name, type, paramName, value) {
      this.sendItemMsg("editParam", "data", { name, type, paramName, value });
    }
  }
}
</script>

<style>
#parent-div {
  padding: 10px;
}
</style>
