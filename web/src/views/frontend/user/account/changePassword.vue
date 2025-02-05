<template>
    <div class="user-views">
        <el-card class="user-views-card" shadow="hover" :header="t('user.user.Change Password')">
            <div class="change-password">
                <el-form :model="state.form" :rules="state.rules" label-position="top" ref="formRef" @keyup.enter="onSubmit(formRef)">
                    <FormItem
                        :label="t('user.user.Old password')"
                        type="password"
                        v-model="state.form.oldPassword"
                        prop="oldPassword"
                        :input-attr="{ 'show-password': true }"
                        :placeholder="t('user.user.Please enter your current password')"
                    />
                    <FormItem
                        :label="t('user.user.New password')"
                        type="password"
                        v-model="state.form.newPassword"
                        prop="newPassword"
                        :input-attr="{ 'show-password': true }"
                        :placeholder="t('Please input field', { field: t('user.user.New password') })"
                    />
                    <FormItem
                        :label="t('user.user.Confirm new password')"
                        type="password"
                        v-model="state.form.confirmPassword"
                        prop="confirmPassword"
                        :input-attr="{
                            'show-password': true,
                        }"
                        :placeholder="t('Please input field', { field: t('user.user.Confirm new password') })"
                    />
                    <el-form-item class="submit-buttons">
                        <el-button @click="onResetForm(formRef)">{{ $t('Reset') }}</el-button>
                        <el-button type="primary" :loading="state.formSubmitLoading" @click="onSubmit(formRef)">{{ $t('Save') }}</el-button>
                    </el-form-item>
                </el-form>
            </div>
        </el-card>
    </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { FormInstance } from 'element-plus'
import { onResetForm } from '/@/utils/common'
import { buildValidatorData } from '/@/utils/validate'
import { changePassword } from '/@/api/frontend/user/index'
import { useI18n } from 'vue-i18n'
import FormItem from '/@/components/formItem/index.vue'
import { useRouter } from 'vue-router'

const { t } = useI18n()

const router = useRouter()
const formRef = ref<FormInstance>()
const state = reactive({
    formSubmitLoading: false,
    form: {
        oldPassword: '',
        newPassword: '',
        confirmPassword: '',
    },
    rules: {
        oldPassword: [buildValidatorData('required', t('user.user.Old password'))],
        newPassword: [buildValidatorData('required', t('user.user.New password')), buildValidatorData('password')],
        confirmPassword: [
            buildValidatorData('required', t('user.user.Confirm new password')),
            buildValidatorData('password'),
            {
                validator: (rule: any, val: string, callback: Function) => {
                    if (state.form.newPassword || state.form.confirmPassword) {
                        if (state.form.newPassword == state.form.confirmPassword) {
                            callback()
                        } else {
                            callback(new Error(t('user.user.The duplicate password does not match the new password')))
                        }
                    }
                    callback()
                },
                trigger: 'blur',
            },
        ],
    },
})

const onSubmit = (formEl: FormInstance | undefined) => {
    if (!formEl) return
    formEl.validate((valid) => {
        if (valid) {
            state.formSubmitLoading = true
            changePassword(state.form)
                .then((res) => {
                    state.formSubmitLoading = false
                    if (res.code == 1) {
                        router.push({ name: 'userLogin' })
                    }
                })
                .catch(() => {
                    state.formSubmitLoading = false
                })
        }
    })
}
</script>

<style scoped lang="scss">
.change-password {
    width: 360px;
}
.submit-buttons :deep(.el-form-item__content) {
    justify-content: flex-end;
}
</style>
