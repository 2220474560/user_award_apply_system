<!--
    * @FileDescription: 报名表单管理 —— 科研之星报名管理。
    * @Author: 蒋雯绘
    * @Date: 2024年2月2日
    * @LastEditors: 蒋雯绘
    * @LastEditTime: 2024年2月2日
-->
<template>
  <div class="contain">
    <a-table class="ant-table-striped" :dataSource="dataSource" :columns="columns" bordered>
      <template #bodyCell="{ column, record }">
        <template v-if="column.dataIndex == 'address'">
          <a-button type="link" @click="showModal3">点击查看</a-button>
          <a-modal
            v-model:open="open3"
            title="佐证材料"
            @ok="handleOk3"
            :ok-button-props="{ disabled: true }"
            :cancel-button-props="{ disabled: true }"
          >
            <p>{{ record.name }}</p>
            <p>{{ record.name }}</p>
            <p>{{ record.name }}</p>
          </a-modal>
        </template>
        <template v-if="column.dataIndex == 'state' && record.state == 1">
          <a style="color: #81c26b" class="aaa">已通过</a>
        </template>
        <template v-if="column.dataIndex == 'state' && record.state == 0">
          <a style="color: #d41212" class="aaa">未通过</a>
        </template>
        <template v-else-if="column.dataIndex == 'operate'">
          <template v-if="record.state == 1">
            <a-button type="link" style="color: green" @click="showModal4">查看证书</a-button>
            <a-modal
              v-model:open="open4"
              title="查看详情"
              @ok="handleOk4"
              :ok-button-props="{ disabled: true }"
              :cancel-button-props="{ disabled: true }"
            >
            </a-modal>
          </template>
          <template v-else-if="record.state == 0">
            <a-button
              type="primary"
              style="background-color: rgb(241, 170, 78)"
              @click="showModal1"
            >
              修改
            </a-button>
            <a-modal v-model:open="open" title="修改填写内容" @ok="handleOk" >
              <!-- 修改表单 -->
              <a-form :model="formState" style="max-width: 500px">
                <a-form-item label="类别">
                  <a-input v-model:value="formState.scitype" />
                </a-form-item>
                <a-form-item label="项目名称/软著名称/期刊名称">
                  <a-input v-model:value="formState.sciname" />
                </a-form-item>
                <a-form-item label="项目级别/颁发单位/论文名称">
                  <a-input v-model:value="formState.scigrade" />
                </a-form-item>
                <a-form-item label="排名/总人数">
                  <a-input v-model:value="formState.ranking" />
                </a-form-item>
                <a-form-item label="立项时间/获批时间/发表时间">
                  <a-input v-model:value="formState.signuptime" />
                </a-form-item>
                <a-form-item label="佐证材料" name="dragger" >
                    <a-upload-dragger
                      v-model:fileList="formState.dragger"
                      name="files"
                      action="/upload.do"
                      :beforeUpload="beforeUpload"
                      @change="handleChange"
                    >
                    <p class="ant-upload-drag-icon">
                      <InboxOutlined />
                    </p>
                    <p class="ant-upload-text">将文件拖到此处或点击上传</p>
                  </a-upload-dragger>
                </a-form-item>
                <a-form-item label="id">
                  <a-input v-model:value="record.key"/>
                </a-form-item>
              </a-form>
            </a-modal>
            <a-popconfirm
              v-if="dataSource.length"
              title="是否想要删除?"
              @confirm="onDelete(record.key, record)"
            >
              <a-button type="primary" style="margin-left: 45px" danger>删除</a-button>
            </a-popconfirm>
          </template>
          <template v-else-if="record.state == 2">
            <a-button type="link" danger @click="showModal2">查看驳回原因</a-button>
            <a-modal
              v-model:open="open2"
              title="查看驳回原因"
              @ok="handleOk2"
              :ok-button-props="{ disabled: true }"
              :cancel-button-props="{ disabled: true }"
            >
              <p>{{ record.name }}</p>
            </a-modal>
            <a-popconfirm
              v-if="dataSource.length"
              title="是否想要删除?"
              @confirm="onDelete(record.key, record)"
            >
              <a-button type="primary" style="margin-left: 45px" danger>删除</a-button>
            </a-popconfirm>
          </template>
        </template>
      </template>
    </a-table>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, reactive, toRaw, type UnwrapRef } from 'vue'
import { message, Upload } from 'ant-design-vue'
import { InboxOutlined } from '@ant-design/icons-vue'
import type { UploadChangeParam } from 'ant-design-vue'
import { JWHgetresearchRequest,JWHeditRequest,JWHdeleteRequest,JWHupfileRequest } from '../../../service/mains/apply-manage/research-star'
//定义表单
interface FormState {
  scitype: string
  sciname:string
  scigrade:string
  ranking:string
  signuptime: string
  materials: string
  state: string
  address:string
  operate: string
  dragger: any[]
}
const formState: UnwrapRef<FormState> = reactive({
  scitype:'',
  sciname:'',
  scigrade:'',
  ranking:'',
  signuptime: '',
  materials: '',
  state: '',
  address:'',
  operate: '',
  dragger: []
})
//表格表头
const columns = [
  {
    title: '类别',
    dataIndex: 'scitype',
    key: 'scitype',
    width: 160,
    align: 'center'
  },
  {
    title: '项目名称/软著名称/期刊名称',
    dataIndex: 'sciname',
    key: 'sciname',
    width: 160,
    align: 'center'
  },
  {
    title: '项目级别/颁发单位/论文名称',
    dataIndex: 'scigrade',
    key: 'scigrade',
    width: 160,
    align: 'center'
  },
  {
    title: '排名/总人数',
    dataIndex: 'ranking',
    key: 'ranking',
    width: 160,
    align: 'center'
  },
  {
    title: '立项时间/获批时间/发表时间',
    dataIndex: 'signuptime',
    key: 'signuptime',
    width: 160,
    align: 'center'
  },
  {
    title: '佐证材料',
    dataIndex: 'address',
    key: 'address',
    width: 160,
    align: 'center'
  },
  {
    title: '状态',
    dataIndex: 'state',
    key: 'state',
    width: 160,
    align: 'center',
  },
  {
    title: '操作',
    dataIndex: 'operate',
    key: 'operate',
    width: 300,
    align: 'center',
  }
]
// 表格内容
const dataSource = ref<{
  key: string;
  scitype: string;
  sciname: string;
  scigrade: string;
  ranking: string;
  signuptime: string;
  state: string;
}[]>([]);
//获取科研表单页面
  grade();
async function grade() {
// 取出登录token
const researchtoken = "bearer"+' '+localStorage.getItem('LOGIN_TOKEN');
    //console.log(doubletoken)
  const researchResult = await JWHgetresearchRequest({
    headers: {
      Authorization: researchtoken,
    },
  })
  if (researchResult.code == 200) {
    console.log(researchResult.data)
    for (let i = 0; i < researchResult.data.length; i++) {
      const item = researchResult.data[i];


      const dataItem = {
        key: item.id,
        scitype: item.scitype,
        sciname: item.sciname,
        scigrade: item.scigrade,
        ranking: item.ranking,
        signuptime: item.signuptime,
        state: item.state,

      };
      dataSource.value.push(dataItem);
    }
  }
}

 //删除后端数据
async function liedelete(key:string) {
  const deleteResult = await JWHdeleteRequest(key)
   console.log(key)
  if (deleteResult.code == 200) {
    message.success(`${deleteResult.msg}`)
  } else {
    message.warning(`${deleteResult.msg}`)
  }
}
const count = computed(() => dataSource.value.length + 1)
//删除前端数据
const onDelete = (key: string, record) => {
  delete dataSource[key]
  dataSource.value = dataSource.value.filter((item) => item.key !== key)
  liedelete(key)
}
//佐证材料
const open3 = ref<boolean>(false)
//上传pdf
const beforeUpload = (file: any) => {
  const isPDF = file.type === 'application/pdf'
  if (!isPDF) {
    message.error('只能上传 PDF 文件！')
    return false;
  }
  upfile(file); // 直接将 file 对象传递给 upfile 函数
  return false; // 返回 false 取消默认的上传行为
}
const handleChange = (info: UploadChangeParam) => {
  const status = info.file.status
  if (status !== 'uploading') {
    console.log(info.file, info.fileList)
  }
  if (status === 'done') {
    message.success(`${info.file.name} file uploaded successfully.`)

  } else if (status === 'error') {
    message.error(`${info.file.name} file upload failed.`)
  }
}
// 取出登录token
const filetoken = "bearer"+' '+localStorage.getItem('LOGIN_TOKEN');
    console.log(filetoken)
async function upfile(file: any) {
  const fileResult = await JWHupfileRequest(file,filetoken)
  if (fileResult.code == 200) {
    message.success(`${fileResult.msg}`)
    localStorage.setItem('fileResult', fileResult.data)
  } else {
    message.warning(`${fileResult.msg}`)
  }

}
const showModal3 = () => {
  open3.value = true
}


const handleOk3 = (e: MouseEvent) => {
  console.log(e)
  open3.value = false
}
//点击修改
const open = ref<boolean>(false)

const showModal1 = () => {
  open.value = true
}

const handleOk = (e: MouseEvent,key:string, record) => {
  // console.log(e)
  open.value = false
  const upfile = localStorage.getItem('fileResult');
  const edit = async () => {
    const editResult = await JWHeditRequest(formState.scitype,formState.sciname,formState.scigrade,formState.ranking,formState.signuptime,upfile,key)
  // console.log(editResult)
  if (editResult.code == 200) {
    message.success(`${editResult.msg}`)
  } else {
    message.warning(`${editResult.msg}`)
  }
  }
  edit();
  grade();
}
// 查看驳回原因
const open2 = ref<boolean>(false)

const showModal2 = () => {
  open2.value = true
}

const handleOk2 = (e: MouseEvent) => {
  console.log(e)
  open2.value = false
}
// 查看证书
const open4 = ref<boolean>(false)

const showModal4 = () => {
  open3.value = true
}

const handleOk4 = (e: MouseEvent) => {
  console.log(e)
  open3.value = false
}
</script>

<style scoped>
.contain {
  width: 100%;
  height: 100%;
  background-color: rgb(255, 255, 255, 0.6);
  border-radius: 20px;
  display: flex;
  justify-content: center;
  text-align: center;
}
:deep(:where(.css-dev-only-do-not-override-19yxfbp).ant-table-wrapper .ant-table-pagination-right) {
  opacity: 0;
}
:deep(:where(.css-dev-only-do-not-override-19yxfbp).ant-table-wrapper .ant-table) {
  margin-top: 40px;
}
:deep(
    :where(.css-dev-only-do-not-override-19yxfbp).ant-table-wrapper
      .ant-table-container
      table
      > thead
      > tr:first-child
      > *:first-child
  ) {
  font-weight: normal;
}
:deep(:where(.css-dev-only-do-not-override-19yxfbp).ant-table-wrapper .ant-table-thead > tr > th) {
  font-weight: normal;
}
</style>
