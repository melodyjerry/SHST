<template>
    <view>
        <headslot title="公告"></headslot>
        <view style="margin-top: 10px;"></view>

        <layout v-for="(item,index) in data" :key="index">
            <rich-text :nodes="item.announce" style="line-height: 23px;"></rich-text>
        </layout>

        <!-- #ifdef MP-WEIXIN -->
        <official-account style='display: block;'></official-account>
        <!-- #endif -->

    </view>
</template>

<script>
    const app = getApp()
    import headslot from "@/components/headslot.vue"
    export default {
        components: {
            headslot
        },
        data() {
            return {
                data: []
            }
        },
        onLoad: async function(options) {
            uni.setStorage({
                key: 'point',
                data: app.globalData.tips
            })
            var res = await app.request({
                load: 2,
                url: app.globalData.url + 'ext/announce',
            })
            if (res.data.info) {
                res.data.info.reverse();
                this.data = res.data.info
            }
        },
        methods: {
            copy(e) {
                uni.setClipboardData({
                    data: e.currentTarget.dataset.copy
                })
            }
        }
    }
</script>

<style>

</style>
