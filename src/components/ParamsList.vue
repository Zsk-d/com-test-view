<template>
    <div style="padding: 5px;">
        <a-list size="small" bordered :data-source="params">
            <template #renderItem="{ item }">
                <a-list-item>
                    <a-form layout="inline">
                        <a-form-item :label="'● ' + item.name">
                            <a-textarea v-if="item.type == 'str'" :auto-size="{ minRows: 1 }" v-model:value="item.value">
                            </a-textarea>
                            <a-input type="number" v-if="item.type == 'int32'" v-model:value="item.value"></a-input>
                        </a-form-item>
                        <a-form-item>
                            <a-button v-if="!item.isChanged" size="small" danger>x</a-button>
                        </a-form-item>
                        <a-form-item>
                            <a-button v-if="item.isChanged" size="small" danger>✓</a-button>
                        </a-form-item>
                        <a-form-item label="类型">
                            <a-select size="small" v-model:value="item.type" style="width: 80px" :options="[{
                                value: 'int32',
                                label: 'int32'
                            }, {
                                value: 'str',
                                label: '字符串'
                            }]" @change="handleSelectChange"></a-select>
                        </a-form-item>
                        <a-form-item label="长度/字节">
                            <a-input type="number" v-if="item.type == 'str'" style="width: 58px"
                                v-model:value="item.length" size="small">
                            </a-input>
                            <label v-if="item.type == 'int32'">{{ item.length }}</label>
                        </a-form-item>
                    </a-form>
                </a-list-item>
            </template>
        </a-list>
    </div>
</template>
<script>
export default {
    data() { return {} },
    mounted() { },
    props: ["type", "params"],
    components: {},
    methods: {
        handleSelectChange(val) {

        }, itemChange(item) {
            item.isChanged = true;
        }
    }
}
</script>
<style>
</style>