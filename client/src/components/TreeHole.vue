<template>
    <div>
        <el-text tag="b" size="large" style="font-size: 18px; font-style: italic;">
            ### 每个人都需要一个树洞，存放那些不可轻易示人的秘密、引而不发的情绪、难以启齿的柔弱。
        </el-text>
        <br />
        <el-divider />
        <el-row v-if="activities.length === 0">
            <el-col :span="24">
                <el-empty description="暂无发贴信息" />
            </el-col>
        </el-row>
        <el-row v-else>
            <el-col :span="24">
                <el-card v-for="activity in activities" :key="activity._id" class="custom-card">
                    <div class="activity-header">
                        <el-avatar :src="activity.avatar" :size="28"></el-avatar>
                        <div class="author-time">
                            <span class="author">@{{ activity.author }}</span>
                            <span class="time">{{ formatTime(activity.time) }} {{ activity.city }}</span>
                        </div>
                    </div>
                    <p></p>
                    <div class="description">{{ activity.description }}</div>
                    <div class="activity-action-buttons">
                        <el-link :underline="false" size="small" type="danger"
                            @click="handleAction(activity._id, 'like')">👍[ {{ activity.like }} ]</el-link>
                        &nbsp;&nbsp;&nbsp;&nbsp;
                        <el-link :underline="false" size="small" type="success"
                            @click="handleAction(activity._id, 'dislike')">👎[ {{ activity.dislike }} ]</el-link>
                    </div>
                </el-card>
            </el-col>
        </el-row>
        <p></p>
        <!-- 发帖表单 -->
        <el-card>
            <el-form :model="activity" ref="activityForm" label-width="auto">
                <el-form-item label="昵称:" prop="author" class="bold-label">
                    <el-input v-model="activity.author" size="small" style="width: 50%;" clearable
                        class="bold-input"></el-input>
                    &nbsp;&nbsp;&nbsp;&nbsp;
                    <div class="avatar-list">
                        <img v-for="(avatar, index) in avatars" :key="avatar.id" :src="avatar.url"
                            @click="selectAvatar(avatar.url, index)"
                            :class="{ 'selected-avatar': index === selectedAvatarIndex }">
                    </div>
                </el-form-item>
                <el-form-item label="内容:" prop="description" class="bold-label">
                    <el-input v-model="activity.description" maxlength="1000" show-word-limit
                        :autosize="{ minRows: 3, maxRows: 6 }" type="textarea" class="bold-input"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button @click="submitActivity()" :loading="submitting" type="primary" style="width: 100%;">
                        点击发布
                    </el-button>
                </el-form-item>
            </el-form>
        </el-card>
    </div>
</template>

<script>
import { h } from 'vue'
import { ElMessage } from 'element-plus'
export default {
    data() {
        return {
            activity: {
                author: '',
                description: '',
                time: new Date().toISOString(),
                'like': 0,
                'dislike': 0,
                avatar: '',
                city: '',
            },
            activities: [],
            submitting: false,
            avatars: [
                { id: 1, url: 'http://47.100.41.212:8080/images/AvatarResource/punk1714.webp' },
                { id: 2, url: 'http://47.100.41.212:8080/images/AvatarResource/punk4633.webp' },
                { id: 3, url: 'http://47.100.41.212:8080/images/AvatarResource/punk4634.webp' },
                { id: 4, url: 'http://47.100.41.212:8080/images/AvatarResource/punk4636.webp' },
                { id: 5, url: 'http://47.100.41.212:8080/images/AvatarResource/punk5822.webp' },
                { id: 6, url: 'http://47.100.41.212:8080/images/AvatarResource/punk.png' },
            ],
            selectedAvatarIndex: 0,
        };
    },
    methods: {
        formatTime(time) {
            const now = new Date();
            const activityTime = new Date(time);
            const timeDiff = now - activityTime;
            const seconds = Math.floor(timeDiff / 1000);
            const minutes = Math.floor(seconds / 60);
            const hours = Math.floor(minutes / 60);
            const days = Math.floor(hours / 24);

            if (days > 0) {
                return `${days}天前`;
            } else if (hours > 0) {
                return `${hours}小时前`;
            } else if (minutes > 0) {
                return `${minutes}分钟前`;
            } else {
                return '刚刚';
            }
        },
        selectAvatar(avatarUrl, index) {
            this.activity.avatar = avatarUrl;
            this.selectedAvatarIndex = index;
        },
        async fetchActivities() {
            this.activities = (await this.$axios.get('/posts')).data;
        },
        async submitActivity() {
            this.$refs.activityForm.validate();
            if (this.activity.author.trim() && this.activity.description.trim()) {
                try {
                    this.submitting = true;
                    delete this.activity.activity_id;

                    const response = await fetch('https://ipinfo.io/json');
                    const data = await response.json();
                    this.activity.city = data.city;

                    await this.$axios.post('/posts', this.activity);
                    this.fetchActivities();
                    this.clearForm();
                    ElMessage.success({
                        message: h('p', { style: 'line-height: 1; font-size: 14px' }, [
                            h('span', null, '发布成功！'),
                        ]),
                    })
                } finally {
                    this.submitting = false;
                }
            } else {
                ElMessage.info({
                    message: h('p', { style: 'line-height: 1; font-size: 14px' }, [
                        h('span', null, '填写完整信息发布！'),
                    ]),
                })
            }
        },
        async handleAction(_id, actionType) {
            await this.$axios.post(`/${actionType}`, { _id });
            this.fetchActivities();
        },
        clearForm() {
            this.activity.description = '';
        },
    },
    mounted() {
        this.fetchActivities();
    },
};
</script>

<style>
.activity-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.action-buttons {
    display: flex;
}

.el-card {
    margin-bottom: 1.5%;
    padding: 0.1px;
    font-size: 15px;
}

.author {
    font-weight: bold;
}

.time {
    margin-left: 5px;
    font-size: 12px;
    color: #888;
}

.activity-header {
    display: flex;
    align-items: center;
}

.author-time {
    margin-left: 5px;
}

.activity-action-buttons {
    display: flex;
    justify-content: flex-end;
}

.bold-label .el-form-item__label {
    font-weight: bold;
}

.description {
    margin-bottom: 0.1px;
}

.avatar-list {
    display: flex;
    justify-content: flex-start;
    align-items: center;
}

.avatar-list img {
    border-radius: 50%;
    /* overflow: hidden; */
}

.avatar-list img:hover {
    border: 1px solid #409EFF;
}

.selected-avatar {
    border: 1px solid #409EFF;
}
</style>
