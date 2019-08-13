<template>
  <div class="mail-box">
    <el-row  type="flex"  class="row-bg" justify="end">
      <el-col :span="4">
        <el-select v-model="sendCount" clearable  placeholder="请选择发送次数">
          <el-option
            v-for="item in sendCountList"
            :key="item.key"
            :label="item.label"
            :value="item.key">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="4">
        <el-select v-model="mailStatusSNMP" clearable  placeholder="请选择发送状态">
          <el-option
            v-for="item in mailStatusList"
            :key="item.key"
            :label="item.label"
            :value="item.key">
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="2">
        <el-button  type="primary" icon="el-icon-search" @click="getmails()">搜索</el-button>
      </el-col>
    </el-row>
    <el-table
      :data="mails"
      style="width: 100%">
      <el-table-column
        prop="mailSubject"
        align="center"
        label="主题 "
        width="230">
      </el-table-column>
      <el-table-column
        prop="fromMail"
        align="center"
        label="发件人"
        width="290">
      </el-table-column>
      <el-table-column
        prop="toMail"
        align="center"
        label="收件人"
        width="290">
      </el-table-column>
      <el-table-column
        sortable
        prop="sendTime"
        align="center"
        label="发送时间"
        width="290">
        <!--        :formatter="dateFormat"-->
      </el-table-column>
      <el-table-column
        sortable
        prop="status"
        align="center"
        label="发送状态"
        width="170">
      </el-table-column>
      <el-table-column
        sortable
        prop="sendNum"
        align="center"
        label="发送次数"
        width="170">
      </el-table-column>
      <el-table-column label="操作" fixed="right" width="170" align="center">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            plain
            @click="handleEdit(scope.$index, scope.row)">查看</el-button>
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page.sync="currentPage"
      :page-sizes="[10, 20, 30, 50]"
      :page-size="pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    <el-dialog title="查看邮件详细信息" :visible.sync="mailFormVisible" top="10vh" @close="resetForm('mailForm')">
      <el-form :model="mail" ref="mailForm" >
        <el-form-item label="标题" prop="mailSubject" label-width="100px">
          <el-input v-model="mail.mailSubject" autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="发件人" label-width="100px">
          <el-input v-model="mail.fromMail" autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="抄送人" label-width="100px">
          <el-input v-model="mail.ccMail" autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="密送人" label-width="100px">
          <el-input v-model="mail.bccMail" autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="收件人" label-width="100px">
          <el-input v-model="mail.toMail" autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="计划发送时间" label-width="100px">
          <el-input v-model="mail.planSendTime" autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="发送时间" label-width="100px">
          <el-input v-model="mail.sendTime"  autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="邮件正文" label-width="100px">
          <el-input v-model="mail.mailContent"  autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="服务器标识" label-width="100px">
          <el-input v-model="mail.serverFlag"  autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="发送状态" label-width="100px">
          <el-input v-model="mail.status"  autocomplete="off" readonly></el-input>
        </el-form-item>
        <el-form-item label="发送次数" label-width="100px">
          <el-input v-model="mail.sendNum"  autocomplete="off" readonly></el-input>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
  // 引入富文本编辑器所需样式
  import 'quill/dist/quill.core.css'
  import 'quill/dist/quill.snow.css'
  import 'quill/dist/quill.bubble.css'
  import moment from 'moment'

  export default {
    data () {
      return {
        mails: [],
        total:0,//默认数据总数
        currentPage:1,//默认开始页面
        pageSize:10,
        mail: {
          id: '',
          sendTime: '',
          mailSubject: '',
          frommail: '',
          tomail: '',
          status: '已发送',
          sendNum:''
        },
        mailStatusList: [
          {
            key: '0',
            label: '计划邮件'
          },
          {
            key: '1',
            label: '已发送'
          },
          {
            key: '2',
            label: '发送失败'
          }
        ],
        mailStatusSNMP:'',
        sendCountList: [
          {
            key: '1',
            label: '一次'
          },
          {
            key: '2',
            label: '两次'
          },
          {
            key: '3',
            label: '三次'
          }
        ],
        sendCount: '',

        mailFormVisible: false,
        rowIndex: 9999,
      }
    },
    components: {
      // quillEditor
    },
    mounted () {
      this.getmails()
    },
    methods: {
      getmails () {
        let postData = this.$qs.stringify({
          page:this.currentPage,
          rows:this.pageSize,
          status: this.mailStatusSNMP,
          sendCount: this.sendCount
        });
        this.loading = true;
        this.$http({method:"post",url:'http://localhost:8086/mail/page',data:postData}).then((res) => {
          console.log(res.data);
          if (res.data == ''){
            this.mails= [];
            this.total=0;
          }else {
            let chan=res.data.items.length;
            for (let i = 0; i < chan; i++) {
              if (res.data.items[i].status==0){
                res.data.items[i].status="待发送";
                res.data.items[i].sendTime=moment(res.data.items[i].sendTime).format("YYYY-MM-DD HH:mm:ss")
              }else if (res.data.items[i].status==1){
                res.data.items[i].status="已发送";
                res.data.items[i].sendTime=moment(res.data.items[i].sendTime).format("YYYY-MM-DD HH:mm:ss")
              } else if (res.data.items[i].status==2) {
                res.data.items[i].status="发送失败";
                res.data.items[i].sendTime=moment(res.data.items[i].sendTime).format("YYYY-MM-DD HH:mm:ss")
              }
            }
            this.mails = res.data.items;
            this.total= res.data.total
          }
        }).catch((err) => {
          console.error(err)
        })
      },

      //时间格式化
      dateFormat:function(row,column){        
        var date = row[column.property];        
        if(date == undefined){return ''};        
        return moment(date).format("YYYY-MM-DD HH:mm:ss")    
      },
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.pageSize=val;
        this.getmails()
      },
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        this.currentPage=val;
        this.getmails();
      },

      handleEdit (index, row) {
        this.mail = Object.assign({}, row)
        this.mailFormVisible = true
        this.rowIndex = index
      },
      editmail (formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            this.mails.splice(this.rowIndex, 1, this.mail)
            this.mailFormVisible = false
            this.$message({
              type: 'success',
              message: '修改成功!'
            })
          }
        })
      },

      handleDelete (index, row) {
        this.$confirm(`确定删除新闻 【${row.mailSubject}】 吗?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          console.log(index,row);
          console.log(row.id);

          let delData = this.$qs.stringify({
            mailId: row.id,
          });
          this.$http({method:"post",url:'http://localhost:8086/mail/delete',data:delData}).then(() => {
            this.mails.splice(index, 1);
            this.$message({
              type: 'success',
              message: '删除成功!'
            })
          }).catch((err) => {
            console.error('删除失败')
          });
        }).catch(() => {
          console.log('取消删除')
        })
      },
      resetForm (formName) {
        this.$refs[formName].clearValidate()
      }
    }
  }
</script>

<style lang="scss" scoped>
  .mail-box {
    width: 100%;
  }
  .el-pagination {
    margin-top: 20px;
  }
  .quill-editor {
    height: 300px;
    margin-bottom: 20px;
  }
</style>
