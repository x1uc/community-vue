<template>
    <div class="notice">
        <el-row class="tac">
            <el-col :span="4">
                <el-menu default-active="2" class="el-menu-vertical-demo">
                    <el-menu-item index="2" @click="getMessage(like)">
                        <el-icon>
                            <TrophyBase />
                        </el-icon>
                        <span>点赞通知</span>
                    </el-menu-item>
                    <el-menu-item index="3" @click="commentHandleCurrent">
                        <el-icon>
                            <document />
                        </el-icon>
                        <span>评论通知</span>
                        <div class="unComment" v-if="unreadMsg != 0">
                            <div>
                                {{ unreadMsg }}
                            </div>
                        </div>
                    </el-menu-item>
                    <el-menu-item index="4">
                        <el-icon>
                            <setting />
                        </el-icon>
                        <span>系统通知</span>
                    </el-menu-item>
                    <div style="height: 200px;"></div>
                </el-menu>
            </el-col>
            <div v-if="nowSpace == 1">
                <div v-for="item in content" :key="item">
                    <el-card style="width: 900px; min-height: 120px;">
                        <div>
                            {{ item.fromUserName }}
                            <span>{{ space }}</span>
                            {{ item.dateTime }}
                        </div>
                        <div style="margin-top: 10px;">
                            给你的文章
                            <a :href="'#/postContent' + item.entityId" style="text-decoration: none;"> {{ item.title
                                }}</a>
                            点赞，希望你能再接再厉，发布优质帖子
                        </div>
                    </el-card>
                </div>
                <div class="demo-pagination-block">
                    <el-pagination v-model:current-page="likeCurrentPage" v-model:page-size="likePageSize"
                        :page-sizes="[10]" :small="small" :disabled="disabled" :background="background"
                        layout="total, sizes, prev, pager, next, jumper" :total="likeTotal"
                        @current-change="likeHandleCurrentChange" />
                </div>
            </div>
            <div v-if="nowSpace == 2">
                <div v-for="item in content" :key="item">
                    <el-card style="width: 900px; min-height: 120px;">
                        <div>
                            {{ item.formUser }}
                            <span>{{ space }}</span>
                            {{ item.dateTime }}
                        </div>
                        <div style="margin-top: 10px;">
                            在文章
                            <a :href="'#/postContent' + item.entityId" style="text-decoration: none;">
                                {{ item.title }} </a>
                            对你进行了回复，内容为：
                        </div>
                        <div style="margin-top: 20px;">
                            {{ item.content }}
                        </div>
                    </el-card>
                </div>
                <div class="demo-pagination-block">
                    <el-pagination v-model:current-page="commentCurrentPage" v-model:page-size="commentPageSize"
                        :page-sizes="[10]" :small="small" :disabled="disabled"
                        layout="total, sizes, prev, pager, next, jumper" :total="commentTotal"
                        @current-change="commentHandleCurrent" />
                </div>
            </div>
        </el-row>
    </div>
</template>

<script setup>
import axios from 'axios';
import { ref, getCurrentInstance } from 'vue';
import { useRoute, useRouter } from 'vue-router';
const { proxy } = getCurrentInstance();
const route = useRoute();
const router = useRouter();

const content = ref();
const space = ref("                   ")
const like = "like"
const comment = "comment";
const nowSpace = ref(1);


const likeCurrentPage = ref(1);
const likePageSize = ref(10);
const likeTotal = ref(0);


const commentCurrentPage = ref(1);
const commentPageSize = ref(10);
const commentTotal = ref(0);

const unreadMsg = ref(0);

const kfc = (time) => { // 请将 "时间戳" 替换为实际的时间戳
    let tm = parseInt(time);
    let date = new Date(tm);

    let year = date.getFullYear();
    let month = (date.getMonth() + 1).toString().padStart(2, '0');
    let day = date.getDate().toString().padStart(2, '0');
    let hours = date.getHours().toString().padStart(2, '0');
    let minutes = date.getMinutes().toString().padStart(2, '0');
    let formattedDateTime = `${year}-${month}-${day} ${hours}:${minutes}`;
    return formattedDateTime;
}
/**
 * 将日期格式化为指定格式
 * @param val  当前时间： 例- new Data();
 * @param pattern  转化的格式 例- yyyy年-MM月-dd日 hh时:mm分:ss秒 
 * @returns 返回格式化后的时间
 */
const formatDate = (val, pattern) => {
    if (!val) {
        return null;
    }
    if (!pattern) {
        pattern = "yyyy-MM-dd hh:mm:ss"
    }
    return new Date(val).format(pattern);
}

const getMessage = (str) => {
    if (str == 'like')
        nowSpace.value = 1;
    else if (str == 'comment')
        nowSpace.value = 2;
    else
        nowSpace.value = 3;

    let token = localStorage.getItem("token");
    axios({
        url: "/api/message/" + str,
        headers: {
            token: token
        },
        data: {
            currentPage: likeCurrentPage.value,
            pageSize: '10'
        },
        method: 'post'
    }).then(res => {
        if (res.data.code != 200) {
            proxy.Message.error("请先登录");
            return;
        }
        content.value = res.data.data.messageList;
        likeTotal.value = res.data.data.count;
    })
}
//getMessage(like);

const likeHandleCurrentChange = () => {
    getMessage('like')
}

const commentHandleCurrent = () => {
    unreadMsg.value = 0;
    let str = 'comment';
    nowSpace.value = 2;
    let token = localStorage.getItem("token");
    axios({
        url: "/api/message/" + str,
        headers: {
            token: token
        },
        data: {
            currentPage: commentCurrentPage.value,
            pageSize: '10'
        },
        method: 'post'
    }).then(res => {
        if (res.data.code != 200) {
            proxy.Message.error("请先登录");
            return;
        }
        content.value = res.data.data.messageList;
        commentTotal.value = res.data.data.count;
    })
}

getMessage(like);



const unreadComment = () => {
    let token = localStorage.getItem("token");
    axios({
        url: "/api/message/unReadComment",
        headers: {
            token: token
        },
        method: 'get'
    }).then((res) => {
        if (res.data.code == 200) {
            unreadMsg.value = res.data.data;
        }
    })
}


unreadComment();



</script>

<style lang="scss" scoped>
.notice {
    height: 100%;
    width: 100%;
}

.unComment {
    height: 20px;
    width: 20px;
    border-radius: 50%;
    background: red;
    line-height: 20px;
    text-align: center;
}
</style>
