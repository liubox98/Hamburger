<template>
    <div>
        <el-alert :title="`-- 我们已经一起度过了 ${formattedElapsedTime}啦！🎈💕`" type="error" effect="dark" :closable="false" />
        <br>
        <div v-if="!isAuthenticated" class="password-container">
            <div class="input-container">
                <input type="password" v-model="password" placeholder="🔒PIN码" class="password-input">
            </div>
            <div class="message">
                <br>
                <p><b>亲爱的访客：👋</b></p>
                <br>
                <p>非常抱歉给您添麻烦，但我希望能够向您解释一下密码保护的原因。</p>
                <p>这个密码是用来保护我们的隐私内容的，里面包含了一些我们的日常分享和个人生活的内容。</p>
                <p>如果您对内容感兴趣，但是没有获得密码，我希望您能够理解并再多等待一些时间。</p>
                <p>可能会有一天我会考虑去掉密码限制，但目前我们仍然希望保持这种私密性。</p>
                <br>
                <p>感谢您的理解和支持。🙏🌟</p>
            </div>
        </div>
        <div v-if="isAuthenticated">
            <el-button type="primary" @click="showFormDialog" size="large">✍️ 记录一下</el-button>
            <p></p>
            <el-dialog v-model="formDialogVisible" :width="awidthVariable" :fullscreen="isMobile"
                :append-to-body="true">

                <span slot="title"></span>
                <el-form :model="activity" ref="activityForm" label-width="auto">
                    <el-form-item prop="description">
                        <el-input v-model="activity.description" type="textarea" placeholder="点击开始输入..." maxlength="999"
                            show-word-limit clearable :autosize="{ minRows: 2, maxRows: 6 }"></el-input>
                    </el-form-item>
                    <el-form-item prop="description">
                        <el-upload @change="handleFileChange" ref="upload" action="/activities" list-type="picture-card"
                            :auto-upload="false" :file-list="selectedFiles" @remove="handleRemove"
                            accept="image/*, video/*">
                            <el-icon>
                                <Plus />
                            </el-icon>
                            <template #file="{ file }">
                                <div>
                                    <img class="el-upload-list__item-thumbnail" :src="file.url" />
                                    <span class="el-upload-list__item-actions">
                                        <!-- 如果不是移动端或者 disabled 为真，则显示预览按钮 -->
                                        <span v-if="!isMobile || disabled" class="el-upload-list__item-preview"
                                            @click="handlePictureCardPreview(file)">
                                            <el-icon><zoom-in /></el-icon>
                                        </span>
                                        <span class="el-upload-list__item-delete"
                                            @click="handleRemove(file)">
                                            <el-icon>
                                                <Delete />
                                            </el-icon>
                                        </span>
                                    </span>
                                </div>
                            </template>
                        </el-upload>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="submitActivity" :loading="submitting">✌️添加</el-button>
                    </el-form-item>
                </el-form>
            </el-dialog>

            <el-dialog v-model="dialogVisible">
                <img w-full :src="dialogImageUrl" />
            </el-dialog>

            <el-dialog v-model="dialogFormVisible" :width="dwidthVariable" :fullscreen="isMobile">
                <div>
                    <el-carousel :interval="5000" arrow="always">
                        <el-carousel-item v-for="imageData in selectedActivity.image" :key="imageData">
                            <el-image :src="getImageUrl(imageData)" alt="" style="width: 100%; height: 100%"
                                fit="contain"></el-image>
                        </el-carousel-item>
                    </el-carousel>

                    <div class="info">
                        <p><b>{{ selectedActivity.action }}</b></p>
                        <p><b>{{ selectedActivity.description }}</b></p>
                        <el-tag class="ml-2" type="success">{{ formattedCurrentDate(selectedActivity.time) }}</el-tag>
                    </div>
                </div>
            </el-dialog>

            <el-row>
                <el-col v-for="activity in activities" :key="activity._id" :span="getColumnSpan">
                    <el-card
                        :body-style="{ padding: '10px', backgroundImage: `url(${getBackgroundImage(activity.time)})` }">
                        <div style="padding: 14px">
                            <span>{{ activity.action }}</span>
                            <div class="bottom">
                                <time class="time">{{ formattedCurrentDate(activity.time) }}</time>
                                <div class="button-container">
                                    <el-button type="primary" plain
                                        @click="showActivityDetails(activity)">详情</el-button>
                                </div>
                            </div>
                        </div>
                    </el-card>
                </el-col>
            </el-row>
        </div>
    </div>
</template>
<script>
export default {
    data() {
        return {
            password: '',
            correctPassword: 'iamliu',
            isAuthenticated: false,
            awidthVariable: '30%',
            dwidthVariable: '30%',
            activity: { description: '', time: new Date().toISOString(), image: [] },
            activities: [],
            startTime: new Date('2023-10-31T22:01:01').getTime(),
            elapsedTime: 0,
            disabled: false,
            dialogVisible: false,
            formDialogVisible: false,
            dialogImageUrl: '',
            selectedFiles: [],
            submitting: false,
            selectedActivity: null,
            dialogFormVisible: false,
            isMobile: false
        };
    },
    computed: {
        getColumnSpan() {
            return window.innerWidth > 768 ? 6 : 12;
        },
        formattedElapsedTime() {
            const days = Math.floor(this.elapsedTime / (3600 * 24));
            const hours = Math.floor((this.elapsedTime % (3600 * 24)) / 3600);
            const minutes = Math.floor((this.elapsedTime % 3600) / 60);
            const seconds = this.elapsedTime % 60;
            return `${days} 天 ${hours} 时 ${minutes} 分 ${seconds} 秒`;
        },
    },
    watch: {
        password(newValue) {
            this.isAuthenticated = newValue === this.correctPassword;
        }
    },
    methods: {
        checkIsMobile() {
            this.isMobile = window.innerWidth < 768;
        },
        async fetchActivities() {
            this.activities = (await this.$axios.get('/activities')).data;
        },
        getBackgroundImage(date) {
            const year = new Date(date).getFullYear();
            const month = String(new Date(date).getMonth() + 1).padStart(2, '0');
            const day = String(new Date(date).getDate()).padStart(2, '0');
            return `url(https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg)`
            return `url(https://example.com/${year}-${month}-${day}.jpg)`;
        },
        async submitActivity() {
            try {
                this.submitting = true;
                this.activity.image = await Promise.all(this.selectedFiles.map(async file => {
                    const base64Data = await this.convertFileToBase64(file.raw);
                    return base64Data
                }));
                await this.$axios.post('/activities', this.activity);
                this.fetchActivities();
                this.clearForm();
            } finally {
                this.submitting = false;
            }
        },
        convertFileToBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result.split(',')[1]);
                reader.onerror = error => reject(error);
            });
        },
        clearForm() {
            this.activity.action = '';
            this.activity.description = '';
            this.activity.image = [];
            this.formDialogVisible = false;
            this.selectedFiles = [];

        },
        updateElapsedTime() {
            setInterval(() => {
                this.elapsedTime = Math.floor((Date.now() - this.startTime) / 1000);
            }, 1000);
        },
        handleFileChange(file, fileList) {
            this.selectedFiles = fileList;
        },
        handleRemove(file) {
            const index = this.selectedFiles.findIndex(selectedFile => selectedFile.uid === file.uid);
            if (index !== -1) {
                this.selectedFiles.splice(index, 1);
            }
        },
        handlePictureCardPreview(file) {
            if (file.url) {
                this.dialogImageUrl = file.url;
                this.dialogVisible = true;
            }
        },
        showFormDialog() {
            this.formDialogVisible = true;
        },
        formattedCurrentDate(time) {
            const date = new Date(time);
            return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')} ${String(
                date.getHours()
            ).padStart(2, '0')}:${String(date.getMinutes()).padStart(2, '0')}`;
        },
        showActivityDetails(activity) {
            this.selectedActivity = activity;
            this.dialogFormVisible = true;
        },
        getImageUrl(imageData) {
            return `data:image/jpeg;base64,${imageData}`;
        },
    },
    mounted() {
        this.fetchActivities();
        this.updateElapsedTime();
        this.checkIsMobile();
    },
};
</script>

<style>
.password-container {
    text-align: left;
    margin-top: 20px;
}

.input-container {
    display: flex;
    justify-content: center;
}

.password-input {
    padding: 5px;
    border: 1px solid #ccc;
    border-radius: 5px;
}

.message p {
    margin-bottom: 10px;
    font-size: 16px;
}

.message p:first-child {
    font-weight: bold;
}

.message p:last-child {
    margin-bottom: 0;
}

.content {
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 5px;
    background-color: #f9f9f9;
}

.no-header .el-dialog__header {
    display: none;
}

.el-carousel__item {
    margin-right: 20px;
}

.content h2 {
    font-size: 20px;
    color: #333;
}


.content p {
    font-size: 16px;
    color: #666;
}
</style>
