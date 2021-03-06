<template>
  <el-container class="field-props-container">
    <el-header class="field-props-header" v-if="!!!showingInDialog">[文本]字段属性设置</el-header>
    <el-main class="field-props-pane">
      <el-form ref="editorForm" :model="fieldProps" :rules="rules" label-position="left"
               label-width="220px" size="mini" @submit.native.prevent>
        <el-form-item label="字段名称" prop="name">
          <el-input v-model="fieldProps.name" :disabled="fieldState !== 1"></el-input>
        </el-form-item>
        <el-form-item label="显示名称" prop="label">
          <el-input v-model="fieldProps.label"></el-input>
        </el-form-item>
        <el-form-item label="最小长度">
          <el-input-number v-model="fieldProps.fieldViewModel.minLength"
                           :min="0" :max="190" style="width: 100%"></el-input-number>
        </el-form-item>
        <el-form-item label="最大长度（不能超过190字符）">
          <el-input-number v-model="fieldProps.fieldViewModel.maxLength"
                           :min="0" :max="190" style="width: 100%"></el-input-number>
        </el-form-item>
        <el-form-item label="字段校验函数(可多选)" prop="fieldViewModel.validators">
          <el-select multiple allow-create filterable default-first-option :popper-append-to-body="false"
                     v-model="fieldProps.fieldViewModel.validators" style="width: 100%">
            <el-option
                    v-for="(vt, vtIdx) in validators"
                    :key="vtIdx"
                    :label="vt.label"
                    :value="vt.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="是否在列表中默认显示">
          <el-radio-group v-model="fieldProps.defaultMemberOfListFlag" style="float: right">
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="是否允许空值">
          <el-radio-group v-model="fieldProps.nullable" style="float: right">
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="新建记录时允许修改字段">
          <el-radio-group v-model="fieldProps.creatable" style="float: right">
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="更新记录时允许修改字段">
          <el-radio-group v-model="fieldProps.updatable" style="float: right">
            <el-radio :label="true">是</el-radio>
            <el-radio :label="false">否</el-radio>
          </el-radio-group>
        </el-form-item>
        <hr style="border: 0;border-top: 1px dotted #cccccc" />
        <el-form-item>
          <el-button type="primary" size="medium" style="width: 120px" @click="saveField">保存字段</el-button>
          <el-button size="medium" v-if="!!showingInDialog" @click="cancelSave">取消</el-button>
        </el-form-item>
      </el-form>
    </el-main>
  </el-container>
</template>

<script>
  import { addField, getField, updateField } from '@/api/system-manager'
  import FieldState from '@/views/system/field-state-variables'
  import {copyObj} from "@/utils/util";
  import {fieldEditorMixin} from "@/views/system/field-editor/field-editor-mixin";

  export default {
    name: "TextWidgetEditor",
    props: {
      entity: String,
      fieldName: String,
      showingInDialog: Boolean,
      fieldState: {
        type: Number,
        default: FieldState.NEW,
      }
    },
    mixins: [fieldEditorMixin],
    data() {
      return {
        fieldProps: {
          'name': '',
          'label': '',
          'type': 'Text',
          'defaultMemberOfListFlag': true,
          'nullable': false,
          'creatable': true,
          'updatable': true,
          'fieldViewModel': {
            'minLength': 0,
            'maxLength': 190,
            'validators': [],
          },
        },

        rules: {
          name: [
            {required: true, message: '请输入字段名称', trigger: 'blur'},
            {pattern: /^[A-Za-z\d]+$/, message: '请以英文字母、数字开头，不能包含中文字符', trigger: 'blur'},
            {min: 2, max: 30, message: '请输入至少两个字符', trigger: 'blur'},
          ],
          label: [
            {required: true, message: '请输入显示名称', trigger: 'blur'},
            {pattern: /^[A-Za-z\d\u4e00-\u9fa5]+[_-]*/, message: '请以中文、英文字母、数字开头，中间可输入下划线或横杠',
              trigger: 'blur'},
            {min: 2, max: 30, message: '请输入至少两个字符', trigger: 'blur'},
          ],
        },

        validators: [
          {value: 'number', label: '数字'},
          {value: 'mobile', label: '手机号码'},
          {value: 'noChinese', label: '禁止中文'},
          {value: 'email', label: '电子邮箱'},
          {value: 'url', label: 'URL网址'},
        ],

      }
    },
    mounted() {
      //mixin
    },
    methods: {
      saveField() {
        let validResult = true
        this.$refs['editorForm'].validate((success) => {
          if (!success) {
            validResult = false
            return false
          }
        })
        if (!validResult) return

        this.fieldProps.type = 'Text'
        if (this.fieldState === FieldState.NEW) {
          this.createNewField()
        } else if (this.fieldState === FieldState.EDIT) {
          this.modifyOldField()
        } else {
          // error
        }
      },

      cancelSave() {
        this.$emit('cancelSave')
      },

    }
  }
</script>

<style lang="scss" scoped>
  @import "@/styles/form-layout/field-editor-common.scss";
</style>
