<template>
    <div v-if="data != null">
        <h1> {{ userName }}</h1>
        <!-- 正文部分 -->
        <div v-html="data"></div>
        <div style="display: flex;" class="icon-like">
            <el-icon :size="50" :color="getInconColor()" @click="updateLike">
                <TrophyBase />
            </el-icon>
            <h1 style="margin-top: 0px;">{{ likes }}</h1>
        </div>
        <div class="bottom">
            <div>
                ------comment area ------
            </div>
            <div class="comment-info">
                <el-input v-model="commentInput" :autosize="{ minRows: 2, maxRows: 4 }" type="textarea"
                    placeholder="Please input" />
                <el-button type="primary" class="comment-button" @click="addPostComment">
                    发布
                </el-button>
            </div>
        </div>

        <div>-----------</div>
        <!-- 评论部分 -->
        <div class="comment-first">
            <div class="comments-info">
                <el-card v-for="item in comment" :key="item">
                    <div class="comment-user-time">
                        <div class="userNameAndTime">
                            <a href="javascript:void(0)">{{ item.userName }} </a>
                            {{ "发表于 " + item.comment.createTime }}
                        </div>
                        <div>
                            <el-button common size="small"
                                @click="replyClick(item.comment, item.comment)">回复</el-button>
                        </div>
                    </div>
                    {{ item.comment.content }}
                    <div>---------------</div>
                    <div class="child" v-for="reply in item.replies" :key="reply">
                        <div>
                            <a href="javascript:void(0)">{{ reply.userName }}</a> 回复 <a href="javascript:void(0)">{{
        reply.targetName }}</a>
                            <span>{{ " " + reply.comment.createTime }}</span>
                            <el-button common size="small"
                                @click="replyClick(reply.comment, item.comment)">回复</el-button>
                        </div>
                        {{ reply.comment.content }}
                    </div>
                </el-card>
            </div>
        </div>

        <!-- 二级评论的dialog -->
        <el-dialog v-model="dialogVisible" title="回复评论" width="30%" :before-close="handleClose">
            <el-input v-model="dialogInput" :autosize="{ minRows: 2, maxRows: 4 }" type="textarea"
                placeholder="评论字数不超过130字" />
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="dialogVisible = false">取消</el-button>
                    <el-button type="primary" @click="replyPostClick">
                        回复
                    </el-button>
                </span>
            </template>
        </el-dialog>
    </div>
</template>

<script setup>
import hljs from 'highlight.js';
import 'highlight.js/styles/atom-one-dark.css'	//样式
import { ref, getCurrentInstance, nextTick } from 'vue';
import { useRoute } from 'vue-router';
import MarkdownIt from 'markdown-it';
import axios from 'axios'
const { proxy } = getCurrentInstance();
const route = useRoute();
const id = route.params.id;
const data = ref();
const commentInput = ref("");
const userName = ref("");
const comment = ref({});
const likes = ref()
const NowLike = ref(false);
//type = 1 代表主体是 文章 
//type = 2 代表主题是 评论 

//回复评论的对话框




const create = () => {
    axios({
        url: "/api/post/content?id=" + id,
        method: 'get',
    }).then((result) => {
        console.log(result);
        const str = (result.data.data.post.content).toString();
        userName.value = result.data.data.userName;
        //userName.value = result.data.data.post.userName.toString();
        comment.value = result.data.data.commentVos;
        likes.value = result.data.data.likeCount;
        const md = new MarkdownIt();
        data.value = md.render(str.toString());
        highlightCode();
    }).catch((err) => {
        console.log(err);
        proxy.Message.error("错误！！");
    });
}
create();



const addPostComment = () => {
    let token = localStorage.getItem("token");
    axios({
        url: "api/comment/add",
        headers: {
            token: token
        },
        method: 'post',
        data: {
            content: commentInput.value,
            entityId: id,
            type: 1
        }
    }).then((res) => {
        if (res.data.code == 200) {
            proxy.Message.success(res.data.msg);
            commentInput.value = "";
            create();
        }
        else {
            proxy.Message.error(res.data.msg);
        }
    })
}

//点击出对话框时 需要获取评论的ID
const dialogEntityId = ref(""); //评论主题的ID
const dialogTargetId = ref(""); // 评论人的ID
const dialogVisible = ref(false);
const dialogMainCommentId = ref("");
const dialogInput = ref("");

const replyClick = (comment, item) => { // comment 代表回复评论Id ，item 代表主评论id
    dialogVisible.value = true;
    dialogEntityId.value = comment.id;
    dialogTargetId.value = comment.userId;
    dialogMainCommentId.value = item.id;
}

const replyPostClick = () => {
    if (dialogInput.length >= 130) {
        proxy.Message.error("评论最大长度为130");
        return;
    }
    let token = localStorage.getItem("token");
    axios({
        method: 'post',
        url: "/api/comment/addChild",
        headers: {
            token: token
        },
        data: {
            entityId: dialogEntityId.value,
            targetId: dialogTargetId.value,
            content: dialogInput.value,
            commentId: dialogMainCommentId.value,
            postId: id,
            type: 2,
        }
    }).then((res) => {
        if (res.data.code == 200) {
            dialogVisible.value = false;
            proxy.Message.success("评论成功");
            create();
        }
        else {
            proxy.Message.error(res.data.msg);
        }
    })
}

const updateLike = () => {
    let token = localStorage.getItem("token");
    axios({
        url: "/api/post/like/" + id,
        method: 'get',
        headers: {
            token: token
        },
    }).then((res) => {
        if (res.data.code == 200) {
            create();
            likeOrNot();
        } else {
            proxy.Message.error(res.data.msg);
        }
        console.log(res);
    })
}

const likeOrNot = () => {
    let token = localStorage.getItem("token");
    axios({
        url: "/api/post/judgeLike/" + id,
        method: 'get',
        headers: {
            token: token
        },
    }).then((res) => {
        console.log(res);
        if (res.data.code == 200) {
            NowLike.value = res.data.data;
        } else {
            console.log(res.data.msg);
        }
    })
}
likeOrNot();

const getInconColor = () => {
    if (NowLike.value == false) {
        return "black";
    } else {
        return "blue";
    }
}

const highlightCode = () => {
    nextTick(() => {
        let codes = document.querySelectorAll("pre code");
        codes.forEach(item => {
            hljs.highlightBlock(item);
        })
    })
}







</script>

<style lang="scss" scoped>
.bottom {
    height: 100px;
}

.comment-info {
    display: flex;
}

.comment-input {
    padding-top: 0px;
    text-align: left;
    width: 400px;
    height: 100px;
}

.comment-button {
    width: 100px;
    height: 50px;
}

.comment-user-time {
    display: flex;
    justify-content: space-between;
}

a {
    text-decoration: none;
}

.icon-like {
    cursor: pointer
}
</style>
