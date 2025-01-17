<template>
    <div class="profile-settings">
        <alert-box :msg="alert"></alert-box>
        <confirm-box :msg="confirm"></confirm-box>
        <h1>个人信息</h1>
        <div class="user-info">
            <div class="avatar-group">
                <div class="avatar-edit">
                    <img :src="getAvatarUrl(user.avatar)" alt="用户头像" class="avatar" />
                    <div class="edit-icon">
                        <img src="../../public/editing.png" @click="triggerFileInput" />
                    </div>
                    <input type="file" ref="fileInput" @change="onFileChange" class="file-input" />
                </div>
            </div>
            <div class="form-group">
                <label>用户ID</label>
                <p>{{ user.id }}</p>
            </div>
            <div class="form-group">
                <label>昵称</label>
                <div class="nickname-edit">
                    <input type="text"
                           v-model="user.nickname"
                           @input="enableSaveButton"
                           class="nickname-input" />
                    <button :disabled="!isSaveEnabled" @click="save">保存修改</button>
                </div>
            </div>
            <div class="form-group">
                <label>注册时间</label>
                <p>{{ formattedRegisterTime }}</p>
            </div>
            <!-- 登出按钮 -->
            <div class="form-group">
                <button class="logout-button" @click="logout">退出登录</button>
            </div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';
    import global from "../global.js";
    import Cookies from 'js-cookie';
    import AlertBox from './AlertBox.vue';
    import ConfirmBox from './ConfirmBox.vue';

    export default {
        name: 'ProfileSettings',
        components: {
            AlertBox,
            ConfirmBox
        },
        data() {
            return {
                user: {
                    id: '',
                    avatar: '',
                    nickname: '',
                    registerTime: ''
                },
                isSaveEnabled: false,
                alert: '',
                confirm: ''
            }
        },
        mounted() {
            const storedId = localStorage.getItem('globalId');
            this.isEditing = false;
            if (global.id == '') {
                this.user.id = storedId;
                global.id = storedId;
            } else {
                this.user.id = global.id;
                localStorage.setItem('globalId', global.id);
            }
            this.fetchUserInfo();
        },
        methods: {
            fetchUserInfo() {
                var token = Cookies.get('token');
                axios.post(`${this.$Url}/api/user/userInfo`, { token: token })
                    .then(response => {
                        this.user = response.data;
                    })
                    .catch(error => {
                        console.error('Error fetching user data:', error);
                        Cookies.remove('token');
                        this.$router.push('/').then(() => {
                            // 刷新登录页面
                            window.location.reload();
                        });
                        alert("token异常，请重新登陆！");
                    });
            },
            enableSaveButton() {
                this.isSaveEnabled = true;
            },
            triggerFileInput() {
                this.$refs.fileInput.click();
            },
            onFileChange(event) {
                const file = event.target.files[0];
                if (file) {
                    let formData = new FormData();
                    formData.append('file', file);
                    formData.append('id', this.user.id);
                    axios.post(`${this.$Url}/api/Image/upload-personal-image`, formData)
                        .then(response => {
                            this.user.avatar = response.data.data;
                            this.confirmNotification('头像上传成功');
                        })
                        .catch(error => {
                            console.error('Error uploading avatar:', error);
                            this.alertNotification('头像上传失败');
                        });
                }
            },
            save() {
                if (this.user.nickname == "") {
                    this.alertNotification("昵称不允许为空，请重新输入");
                    return;
                }
                axios.post(`${this.$Url}/api/user/updateUserNickname`, {
                    id: this.user.id,
                    newnickname: this.user.nickname
                })
                    .then(() => {
                        console.log('User nickname updated successfully');
                        this.confirmNotification('昵称修改成功');
                        this.isSaveEnabled = false;
                    })
                    .catch(error => {
                        console.error('Error updating user nickname:', error);
                    });
            },
            logout() {
                var token = Cookies.get('token');
                var formData = new FormData();
                formData.append('token', token);
                axios.post(`${this.$Url}/api/user/logout`, formData)
                    .then(() => {
                        // 显示登出成功提示
                        alert("您已成功登出");
                        Cookies.remove('token');
                        this.$router.push('/').then(() => {
                          // 刷新登录页面
                          window.location.reload();
                        });
                    })
                    .catch(error => {
                        console.error('Error when logout:', error);
                    });

                
            },
            alertNotification(message) {
                this.alert = message;
            },
            confirmNotification(message) {
                this.confirm = message;
            },
            getAvatarUrl(avatarPath) {
                if (avatarPath) {
                    return `${this.$Url}${avatarPath}`;
                }
                return '../../public/default.png'; // 默认头像路径
            }
        },
        computed: {
            formattedRegisterTime() {
                if (this.user.registerTime) {
                    return this.user.registerTime.replace('T', ' ');
                }
                return '';
            }
        }
    };
</script>

<style scoped>
    .profile-settings {
        max-width: 500px;
        min-height: 90px;
        margin: 0 auto;
        padding: 32px;
        background-color: #fff;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }

    h1 {
        margin-bottom: 20px;
        font-size: 30px;
        color: #333;
        text-align: center;
        border: 3px solid #F3C7BA;
        border-radius: 8px;
    }

    .user-info {
        display: flex;
        flex-direction: column;
        gap: 20px;
    }

    .form-group {
        display: flex;
        flex-direction: column;
        color: black;
    }

    .avatar-group {
        text-align: center;
        max-width: 240px;
        padding: 15px;
        border: 1px solid #F3C7BA;
        border-radius: 8px;
        box-shadow: 0 10px 10px rgba(0, 0, 0, 0.1);
    }

    label {
        margin-bottom: 5px;
        font-weight: bold;
        font-size: 20px;
        color: #333;
    }

    input[type="text"], p {
        padding: 10px;
        border: 2px solid #ebebeb;
        border-radius: 4px;
        font-size: 16px;
    }

    .nickname-edit {
        display: flex;
        gap: 10px;
    }

    .nickname-input:disabled {
        background-color: #f9f9f9;
    }

    .avatar-edit {
        display: flex;
        align-items: center;
        gap: 10px;
        position: relative; /* 使阴影效果相对于父元素 */
    }

    .avatar {
        width: 160px;
        height: 160px;
        border-radius: 50%;
        object-fit: cover;
    }

    .file-input {
        display: none;
    }

    button {
        padding: 10px 20px;
        background-color: #fbeaea;
        font-size: 15px;
        color: #F8887D;
        border: 3px solid #FADAD6;
        border-radius: 10px;
        cursor: pointer;
        transition: background-color 0.3s, transform 0.3s, color 0.3s;
    }

        button:disabled {
            cursor: not-allowed;
        }

        button:hover:enabled {
            background-color: #ffe5e5;
            transform: scale(1.05);
            color: #F8887D;
            transition: background-color 0.3s, transform 0.3s, color 0.3s;
        }

    .edit-icon {
        cursor: pointer;
        display: flex;
        align-items: center;
    }

        .edit-icon img {
            width: 24px; /* 调整图标大小 */
            height: 24px; /* 调整图标大小 */
            transition: box-shadow 0.3s ease, filter 0.3s ease;
        }

        .edit-icon:hover img {
            box-shadow: 0 1px 1px rgba(0, 0, 0, 0.3);
            filter: brightness(1.1);
        }

    /* 登出按钮样式 */
    .logout-button {
        width: 100%;
        padding: 10px;
        background-color: #F3C7BA;
        font-size: 16px;
        color: white;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        text-align: center;
        transition: background-color 0.3s, transform 0.3s, color 0.3s;
    }

        .logout-button:hover {
            background-color: #fbb1a2;
        }
</style>
