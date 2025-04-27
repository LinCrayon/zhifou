<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

const topstorylist = ref([]);
const dialogVisible = ref(false);
const router = useRouter();

// 获取文章列表数据
async function fetchArticles() {
    try {
        const response = await axios.post('http://127.0.0.1:8088/v1/article/articles', {
            cursor: 0,
            page_size: 10,
            sort_type: 1,
            article_id: 1
        }, {
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MzI0NjM0MDAsImlhdCI6MTcyOTg3MTQwMCwidXNlcklkIjowfQ.4GPgE4O_2e1xTvE4g0JYNeizbEBYO_dzIPMufqcyH-E`, // 添加 access token
            }
        });
        const articles = response.data.articles.map(article => ({
            img: '/src/assets/image/pic5.png',
            name: article.title,
            articleitem: article.content,
            id: article.id,
            favour: article.like_num,
            dislike: article.dislike_num || 0,
            liked: false,
            disliked: false,
            comments: article.comment_num || 0 // 假设后端返回 comment_num
        }));
        topstorylist.value = articles;
    } catch (error) {
        console.error('Error fetching articles:', error);
    }
}

function toggleLike(article) {
    if (article.liked) {
        article.favour -= 1;
        article.liked = false;
    } else {
        article.favour += 1;
        article.liked = true;

        if (article.disliked) {
            article.dislike -= 1;
            article.disliked = false;
        }
    }
}

function toggleDislike(article) {
    if (article.disliked) {
        article.dislike -= 1;
        article.disliked = false;
    } else {
        article.dislike += 1;
        article.disliked = true;

        if (article.liked) {
            article.favour -= 1;
            article.liked = false;
        }
    }
}

function goDetails(v) {
    router.push('/details');
}

onMounted(() => {
    fetchArticles();
});
</script>

<template>
<div class="Topstory-content">
    <ul>
        <li v-for="v in topstorylist" :key="v.id">
            <div @click="goDetails(v)">
                <div class="topstory-hd">
                    <img :src="v.img" alt="">
                    <span>{{v.name}}</span>
                </div>
                <h2 class="topstory-title">{{v.title}}</h2>
                <div class="topstory-articleitem">{{v.articleitem}}</div>
            </div>
            <div class="topstory-actions">
                <ul>
                    <li @click.stop="toggleLike(v)">
                        <i class="iconfont icon-xiangshang1"></i>赞成{{v.favour}}
                    </li>
                    <li @click.stop="toggleDislike(v)">
                        <i class="iconfont icon-xiangxia2"></i>点踩{{v.dislike}}
                    </li>
                    <li>
                        <i class="iconfont icon-xiaoxi"></i>
                        <span>{{v.comments}}条评论</span>
                    </li>
                    <li>
                        <i class="iconfont icon-fenxiang"></i>
                        <span>分享</span>
                    </li>
                    <li @click.stop="dialogVisible = true">
                        <i class="iconfont icon-shoucang"></i>
                        <span>收藏</span>
                    </li>
                    <li :class="v.liked ? 'approve-like' : ''" @click.stop="toggleLike(v)">
                        <i class="iconfont icon-icon-"></i>
                        <span>{{v.liked ? '取消喜欢' : '喜欢'}}</span>
                    </li>
                </ul>
            </div>
        </li>
    </ul>
    <!-- 对话框 -->
    <el-dialog v-model="dialogVisible" title="添加收藏" width="30%">
        <span>你可以创建多个收藏夹，将答案分类收藏</span>
        <template #footer>
            <span class="dialog-footer">
                <el-button @click="dialogVisible = false">取消</el-button>
                <el-button type="primary" @click="dialogVisible = false">收藏</el-button>
            </span>
        </template>
    </el-dialog>
</div>
</template>

<style scoped>
/* 可以添加样式来区分点赞和点踩按钮 */
</style>
