<template>
    <view>

        <view>
            <headslot title="待办事项"></headslot>
            <view style="margin-top: 10px;"></view>
            <layout v-for="(item,index) in todoList" :key="index">
                <view class='y-CenterCon unitTodo' style="justify-content: space-between;">
                    <view>
                        <view class="y-CenterCon" style="margin: 5px 0;">
                            <view class="a-dot" :style="{'background':item.color,'margin':'0 6px 0 3px'}"></view>
                            <view>{{item.event_content}}</view>
                        </view>
                        <view class="y-CenterCon">
                            <view style="margin: 3px;">{{item.todo_time}}</view>
                        </view>
                    </view>
                    <view class="y-CenterCon">
                        <i class='iconfont icon-banner setStatus' @tap='setStatus' :data-id="item.id" :data-index="index"></i>
                        <i class='iconfont icon-x setStatus' @tap='deleteUnit' :data-id="item.id" :data-index="index"></i>
                    </view>
                </view>
            </layout>
            <layout v-if="tips">
                <view class="y-CenterCon">
                    <view class="a-dot" style="background: #EEEEEE;margin-right: 6px;"></view>
                    <view>{{tips}}</view>
                </view>
            </layout>
        </view>

    </view>
</template>

<script>
    const app = getApp()
    const md5 = require('@/utils/md5.js');
    const util = require('@/utils/util.js');
    const pubFct = require('@/vector/pubFct.js');
    import headslot from "@/components/headslot.vue"
    export default {
        components: {
            headslot
        },
        data() {
            return {
                addContent: "",
                dataDo: util.formatDate(), //默认起始时间  
                dataEnd: util.formatDate(), //默认结束时间 
                todoList: [],
                clickFlag: 1,
                tips: "",
                count: 0
            }
        },
        onLoad: async function(options) {
            var endTime = new Date();
            endTime.addDate(1);
            this.dataEnd = util.formatDate("yyyy-MM-dd", endTime);
            if (app.globalData.openid === "") {
                this.tips = "未正常获取用户信息";
            } else {
                var res = await app.request({
                    load: 2,
                    url: app.globalData.url + "todo/getFinEvent",
                })
                if (res.data.data) {
                    if (res.data.data.length === 0) {
                        this.tips = "暂无已完成事项"
                        return;
                    }
                    var curData = util.formatDate();
                    res.data.data.map(function(value) {
                        var diff_color = pubFct.todoDateDiff(curData, value.todo_time, value.event_content);
                        value.diff = diff_color[0];
                        value.color = diff_color[1];
                        return value;
                    })
                    console.log(res.data.data);
                    this.todoList = res.data.data
                    this.count = res.data.data.length
                }
            }
        },
        methods: {
            setStatus: async function(e) {
                var [err, choice] = await uni.showModal({
                    title: '提示',
                    content: '确定标记为未完成吗',
                })
                var index = e.currentTarget.dataset.index;
                var id = e.currentTarget.dataset.id;
                var res = await app.request({
                    url: app.globalData.url + "todo/setNoFinStatus",
                    method: "POST",
                    data: {
                        id: id
                    },
                })
                app.toast("标记成功");
                this.todoList.splice(index, 1);
                this.todoList = this.todoList
                this.tips = this.todoList.length === 0 ? "暂无已完成事项" : ""
                this.count = this.count - 1
            },
            deleteUnit: async function(e) {
                var [err, choice] = await uni.showModal({
                    title: '提示',
                    content: '确定标记为未完成吗',
                })
                if (choice.confirm) {
                    var index = e.currentTarget.dataset.index;
                    var id = e.currentTarget.dataset.id;
                    var res = await app.request({
                        url: app.globalData.url + "todo/deleteUnit",
                        method: "POST",
                        data: {
                            id: id
                        },
                    })
                    app.toast("删除成功");
                    this.todoList.splice(index, 1);
                    this.todoList = this.todoList
                    this.tips = this.todoList.length === 0 ? "暂无已完成事项" : ""
                    this.count = this.count - 1
                }
            }
        },
    }
</script>

<style>
    .a-input {
        border-bottom: 1px solid #eee;
        padding: 5px 0;
    }

    .a-dot {
        margin: 0 3px;
    }

    .unitTable,
    .unitTodo {
        color: #555555;
    }

    .setStatus {
        color: #555555;
        border: 1px solid #EEEEEE;
        padding: 7px;
        border-radius: 20px;
        margin: 0 3px;
    }

    .btn {
        padding: 0 6px;
        border-radius: 1px;
    }
</style>
