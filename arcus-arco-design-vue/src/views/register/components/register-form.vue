<template>
  <div class="register-form-wrapper">
    <div class="register-form-title">{{ $t('register.form.title') }}</div>
    <div class="register-form-sub-title">
      {{ $t('register.form.subTitle') }}
    </div>
    <div class="register-form-error-msg">{{ errorMessage }}</div>
    <a-form
      ref="registerForm"
      :model="userInfo"
      class="register-form"
      layout="vertical"
      @submit="handleSubmit"
    >
      <!-- 头像上传 -->
      <a-form-item
        field="avatar"
        :label="$t('register.form.avatar.label')"
        class="avatar-upload-item"
      >
        <a-upload
          :custom-request="handleAvatarUpload"
          :show-file-list="false"
          accept="image/*"
        >
          <template #upload-button>
            <div class="avatar-upload-wrapper">
              <a-avatar :size="80" class="avatar-preview">
                <img
                  v-if="userInfo.avatar"
                  :src="userInfo.avatar"
                  alt="avatar"
                />
                <icon-camera v-else :size="24" />
              </a-avatar>
              <div class="avatar-upload-text">
                {{ $t('register.form.avatar.placeholder') }}
              </div>
            </div>
          </template>
        </a-upload>
      </a-form-item>

      <!-- 用户名 -->
      <a-form-item
        field="username"
        :rules="[
          { required: true, message: $t('register.form.username.errMsg') },
          { minLength: 3, message: $t('register.form.username.minLength') },
          {
            maxLength: 20,
            message: $t('register.form.username.maxLength'),
          },
        ]"
        :validate-trigger="['change', 'blur']"
        hide-label
      >
        <a-input
          v-model="userInfo.username"
          :placeholder="$t('register.form.username.placeholder')"
        >
          <template #prefix>
            <icon-user />
          </template>
        </a-input>
      </a-form-item>

      <!-- 邮箱 -->
      <a-form-item
        field="email"
        :rules="[
          { required: true, message: $t('register.form.email.errMsg') },
          { type: 'email', message: $t('register.form.email.format') },
        ]"
        :validate-trigger="['change', 'blur']"
        hide-label
      >
        <a-input
          v-model="userInfo.email"
          :placeholder="$t('register.form.email.placeholder')"
        >
          <template #prefix>
            <icon-email />
          </template>
        </a-input>
      </a-form-item>

      <!-- 手机号 -->
      <a-form-item
        field="phone"
        :rules="[
          { required: true, message: $t('register.form.phone.errMsg') },
          {
            match: /^1[3-9]\d{9}$/,
            message: $t('register.form.phone.format'),
          },
        ]"
        :validate-trigger="['change', 'blur']"
        hide-label
      >
        <a-input
          v-model="userInfo.phone"
          :placeholder="$t('register.form.phone.placeholder')"
        >
          <template #prefix>
            <icon-phone />
          </template>
        </a-input>
      </a-form-item>

      <!-- 密码 -->
      <a-form-item
        field="password"
        :rules="[
          { required: true, message: $t('register.form.password.errMsg') },
          {
            minLength: 6,
            message: $t('register.form.password.minLength'),
          },
        ]"
        :validate-trigger="['change', 'blur']"
        hide-label
      >
        <a-input-password
          v-model="userInfo.password"
          :placeholder="$t('register.form.password.placeholder')"
          allow-clear
        >
          <template #prefix>
            <icon-lock />
          </template>
        </a-input-password>
      </a-form-item>

      <!-- 确认密码 -->
      <a-form-item
        field="confirmPassword"
        :rules="[
          {
            required: true,
            message: $t('register.form.confirmPassword.errMsg'),
          },
          { validator: validateConfirmPassword },
        ]"
        :validate-trigger="['change', 'blur']"
        hide-label
      >
        <a-input-password
          v-model="userInfo.confirmPassword"
          :placeholder="$t('register.form.confirmPassword.placeholder')"
          allow-clear
        >
          <template #prefix>
            <icon-lock />
          </template>
        </a-input-password>
      </a-form-item>

      <a-space :size="16" direction="vertical">
        <!-- 用户协议 -->
        <div class="register-form-agreement">
          <a-checkbox
            v-model="userInfo.agreeTerms"
            :rules="[
              {
                required: true,
                message: $t('register.form.agreement.errMsg'),
              },
            ]"
          >
            {{ $t('register.form.agreement.prefix') }}
            <a-link>{{ $t('register.form.agreement.terms') }}</a-link>
            {{ $t('register.form.agreement.and') }}
            <a-link>{{ $t('register.form.agreement.privacy') }}</a-link>
          </a-checkbox>
        </div>

        <!-- 注册按钮 -->
        <a-button type="primary" html-type="submit" long :loading="loading">
          {{ $t('register.form.register') }}
        </a-button>

        <!-- 返回登录 -->
        <a-button
          type="text"
          long
          class="register-form-login-btn"
          @click="goToLogin"
        >
          {{ $t('register.form.backToLogin') }}
        </a-button>
      </a-space>
    </a-form>
  </div>
</template>

<script lang="ts" setup>
  import { ref, reactive } from 'vue';
  import { useRouter } from 'vue-router';
  import { Message } from '@arco-design/web-vue';
  import { ValidatedError } from '@arco-design/web-vue/es/form/interface';
  import { useI18n } from 'vue-i18n';
  import useLoading from '@/hooks/loading';

  interface RegisterData {
    username: string;
    email: string;
    phone: string;
    password: string;
    confirmPassword: string;
    avatar: string;
    agreeTerms: boolean;
  }

  const router = useRouter();
  const { t } = useI18n();
  const errorMessage = ref('');
  const { loading, setLoading } = useLoading();
  // const userStore = useUserStore(); // 暂时注释掉，因为还没有实现注册功能

  const userInfo = reactive<RegisterData>({
    username: '',
    email: '',
    phone: '',
    password: '',
    confirmPassword: '',
    avatar: '',
    agreeTerms: false,
  });

  // 验证确认密码
  const validateConfirmPassword = (
    value: string,
    callback: (error?: string) => void
  ) => {
    if (value !== userInfo.password) {
      callback(t('register.form.confirmPassword.notMatch'));
    } else {
      callback();
    }
  };

  // 处理头像上传
  const handleAvatarUpload = (option: any) => {
    return new Promise<void>((resolve, reject) => {
      const { fileItem } = option;
      const { file } = fileItem;

      if (!file) {
        resolve();
        return;
      }

      const reader = new FileReader();

      reader.onload = (e) => {
        try {
          userInfo.avatar = e.target?.result as string;
          resolve();
        } catch (error) {
          reject(error);
        }
      };

      reader.onerror = () => {
        reject(new Error('Failed to read file'));
      };

      reader.readAsDataURL(file);
    });
  };

  // 处理表单提交
  const handleSubmit = async ({
    errors,
    values,
  }: {
    errors: Record<string, ValidatedError> | undefined;
    values: Record<string, any>;
  }) => {
    if (loading.value) return;

    if (!values.agreeTerms) {
      Message.error(t('register.form.agreement.errMsg'));
      return;
    }

    if (!errors) {
      setLoading(true);
      try {
        // 这里调用注册 API，需要根据实际后端接口调整
        // await userStore.register(values as RegisterData);

        // 模拟注册成功
        await new Promise((resolve) => {
          setTimeout(resolve, 1000);
        });

        Message.success(t('register.form.register.success'));

        // 使用 nextTick 确保 DOM 更新完成后再进行路由跳转
        await new Promise((resolve) => {
          setTimeout(resolve, 100);
        });

        // 注册成功后跳转到登录页面
        await router.push('/login');
      } catch (err) {
        errorMessage.value = (err as Error).message;
      } finally {
        setLoading(false);
      }
    }
  };

  // 返回登录页面
  const goToLogin = async () => {
    try {
      await router.push('/login');
    } catch (error) {
      // Silent navigation error handling
    }
  };
</script>

<style lang="less" scoped>
  .register-form {
    &-wrapper {
      width: 380px;
    }

    &-title {
      color: var(--color-text-1);
      font-weight: 500;
      font-size: 24px;
      line-height: 32px;
      margin-bottom: 4px;
    }

    &-sub-title {
      color: var(--color-text-3);
      font-size: 16px;
      line-height: 24px;
      margin-bottom: 20px;
    }

    &-error-msg {
      height: 32px;
      color: rgb(var(--red-6));
      line-height: 32px;
    }

    &-agreement {
      font-size: 14px;
      color: var(--color-text-2);
    }

    &-login-btn {
      color: var(--color-text-3) !important;
    }
  }

  .avatar-upload-item {
    text-align: center;
    margin-bottom: 24px;

    .avatar-upload-wrapper {
      display: flex;
      flex-direction: column;
      align-items: center;
      cursor: pointer;
    }

    .avatar-preview {
      border: 2px dashed var(--color-border-2);
      transition: border-color 0.3s;

      &:hover {
        border-color: rgb(var(--primary-6));
      }

      img {
        width: 100%;
        height: 100%;
        object-fit: cover;
      }
    }

    .avatar-upload-text {
      margin-top: 8px;
      font-size: 14px;
      color: var(--color-text-3);
    }
  }
</style>
