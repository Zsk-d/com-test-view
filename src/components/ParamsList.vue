<template>
    <div style="padding: 5px;">
        <a-list size="small" bordered :data-source="params" :locale="{ emptyText: '无字段数据' }">
            <template #renderItem="{ item }">
                <a-list-item>
                    <a-form layout="inline">
                        <a-form-item :label="item.name">
                            <a-textarea v-if="item.type == 'str'" :auto-size="{ minRows: 1 }" v-model:value="item.value"
                                @blur="valueBulr(item.name, item.value)" :allowClear="true">
                            </a-textarea>
                            <a-input style="width:170px" type="number" v-if='["int32", "byte"].indexOf(item.type) != -1'
                                @blur="valueBulr(item.name, item.value)" v-model:value="item.value"
                                :maxlength="item.type == 'str' ? parseInt(item.length) : null"></a-input>
                        </a-form-item>
                        <a-form-item>
                            <label>{{ { "str": "字符串", "int32": "int32", "byte": "字节" }[item.type] }}</label>
                        </a-form-item>
                        <a-form-item v-if="item.type == 'str'">
                            <label>{{ { "left": "左填", "right": "右填", "": "左填" }[item.fill] +
                                    `0x${item.fillValue ? item.fillValue : 0}`
                            }}</label>
                        </a-form-item>
                        <a-form-item>
                            <label>{{ item.length + '字节' }}</label>
                        </a-form-item>
                    </a-form>
                </a-list-item>
            </template>
            <a-collapse :bordered="false">
                <a-collapse-panel header="添加字段" :style="{ border: 0 }">
                    <a-form>
                        <a-form-item label="参数名">
                            <a-input style="width:200px" :allowClear="true" v-model:value="param.name">
                            </a-input>
                        </a-form-item>
                        <a-form-item label="参数类型">
                            <a-radio-group v-model:value="param.type" button-style="solid" @change="paramTypeChange">
                                <a-radio-button value="str">字符串</a-radio-button>
                                <a-radio-button value="byte">字节DEC</a-radio-button>
                                <a-radio-button value="int32">int32</a-radio-button>
                            </a-radio-group>
                        </a-form-item>
                        <a-form-item label="长度/字节" v-if="param.type == 'str'">
                            <a-input style="width:200px" type="number" :allowClear="true" v-model:value="param.length">
                            </a-input>
                        </a-form-item>
                        <a-form-item label="填充方向" v-if="param.type == 'str'">
                            <a-radio-group v-model:value="param.fill" button-style="solid">
                                <a-radio-button value="">默认右填0x0</a-radio-button>
                                <a-radio-button value="right">右填</a-radio-button>
                                <a-radio-button value="left">左填</a-radio-button>
                            </a-radio-group>
                        </a-form-item>
                        <a-form-item label="端" v-if="param.type == 'int32'">
                            <a-radio-group v-model:value="param.loc" button-style="solid">
                                <a-radio-button value="big">大端</a-radio-button>
                                <a-radio-button value="small">小端</a-radio-button>
                            </a-radio-group>
                        </a-form-item>
                        <a-form-item label="填充值(16进制ascii)" v-if="param.type == 'str' && param.fill != ''">
                            <a-input :addon-before="'0x'" :allowClear="true" v-model:value="param.fillValue"
                                type="number" style="width:150px"></a-input>
                        </a-form-item>
                        <a-form-item label="初始值">
                            <a-input style="width:200px" :disabled="param.type == 'str' && !param.length"
                                :type="param.type == 'str' ? 'text' : 'number'" show-count
                                :maxlength="param.type == 'str' ? parseInt(param.length) : null" :allowClear="true"
                                v-model:value="param.value">
                            </a-input>
                        </a-form-item>
                        <a-form-item>
                            <a-button type="primary" @click="addParam">添加</a-button>
                        </a-form-item>
                    </a-form>
                </a-collapse-panel>
            </a-collapse>
        </a-list>
    </div>
</template>
<script>
export default {
    data() { return { param: { type: "str", fill: "",loc:"big" } } },
    mounted() { },
    props: ["type", "params", "name", "addParamModalOk", "changeValue"],
    components: {},
    methods: {
        itemChange(item) {
            item.isChanged = true;
        }, paramTypeChange() {
            this.param.type == 'int32' ? this.param.length = 4 : this.param.type == 'byte' ? this.param.length = 1 : this.param.length = null;
        }, addParam() {
            this.addParamModalOk(this.name, this.param, this.type);
        }, valueBulr(paramName, paramValue) {
            this.changeValue(this.name, this.type, paramName, paramValue);
        }
    }
}
</script>
<style>
</style>