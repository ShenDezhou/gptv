<template>
  <div class="content">
    <el-row>
      <el-col :span='24'>
        <el-input
          type='textarea'
          placeholder='请输入内容'
          v-model='prompt'
          maxlength='1024'
          show-word-limit
        >
        </el-input>

      </el-col>
    </el-row>
    <el-row :gutter='20'>
      <el-col :span='18'>
        <el-collapse v-model='activeNames' @change='handleChange'>
          <el-collapse-item title='请求参数' name='1'>
            <el-card :body-style='{ padding: "0px" }' :span='4'>
            <div style='padding: 14px;'>
              <span>GPT引擎</span>
              <div class='bottom clearfix'>
                <el-dropdown split-button type='primary' @click='handleClick' @command="handleCommand">
                  <span>{{engine}}</span>
                  <el-dropdown-menu slot='dropdown'>
                    <el-dropdown-item command="a">CPM</el-dropdown-item>
                    <el-dropdown-item command="b">RPM</el-dropdown-item>
                    <el-dropdown-item command="c">GPT-Neo</el-dropdown-item>
                    <el-dropdown-item command="d">RPM-G2</el-dropdown-item>
                  </el-dropdown-menu>
                </el-dropdown>
              </div>
            </div>
            </el-card>
            <el-card :body-style='{ padding: "0px" }' :span='4'>
            <div style='padding: 14px;'>
              <span>Number</span>
              <div class='bottom clearfix'>
                <el-slider
                  v-model='number'
                  show-stops
                  :max="50"
                  :min="1"
                  show-input>
                </el-slider>
              </div>
            </div>
            </el-card>
            <el-card :body-style='{ padding: "0px" }' :span='4'>
            <div style='padding: 14px;'>
              <span>Response length</span>
              <div class='bottom clearfix'>
                <el-slider
                  v-model='response_l'
                  :max="1024"
                  :min="1"
                  show-input>
                </el-slider>
              </div>
            </div>
            </el-card>
            <el-card :body-style='{ padding: "0px" }' :span='4'>
            <div style='padding: 14px;'>
              <span>Top P</span>
              <div class='bottom clearfix'>
                <el-slider
                  v-model='top_p'
                  :max="1"
                  :min="0"
                  :step="0.001"
                  show-input>
                </el-slider>
              </div>
            </div>
            </el-card>
            <el-card :body-style='{ padding: "0px" }' :span='4'>
            <div style='padding: 14px;'>
              <span>Temperature</span>
              <div class='bottom clearfix'>
                <el-slider
                  v-model='temperature'
                  :max="1"
                  :min="0"
                  :step="0.001"
                  show-input>
                </el-slider>
              </div>
            </div>
            </el-card>
          </el-collapse-item>
        </el-collapse>
      </el-col>
      <el-col :span='6'>
        <el-button type='primary' icon='el-icon-search'
          v-on:click="request">请求</el-button>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span='8'>
        <div class='grid-content bg-purple'></div>
      </el-col>
      <el-col :span='8'>
        <div class='grid-content bg-purple-light'></div>
      </el-col>
      <el-col :span='8'>
        <div class='grid-content bg-purple'></div>
      </el-col>
    </el-row>
    <el-row :gutter='20'>
      <el-col :span='6'>
        GPT输入
        <el-input
          type='textarea'
          placeholder=''
          :autosize='{ minRows: 2, maxRows: 4}'
          v-model='prompt'
          maxlength='1024'
          show-word-limit
        >
        </el-input>
      </el-col>

      <el-col :span='18'>
        <el-table
          :data='tableData'
          style='width: 100%'
          max-height='250'>
          <el-table-column
            prop='lines'
            label='生成'>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { Loading } from 'element-ui'
import axios from 'axios'

export default {
  name: 'Main',
  comments: {},
  data () {
    console.log('url--info', this.$route.path)
    var engine = this.$route.path.toUpperCase().replace('/', '')
    if (engine.length === 0) {
      engine = 'CPM'
    }
    return {
      backends: {
        'CPM': 'http://121.89.205.93:8001/z',
        'RPM': 'http://39.98.127.31:8010/z',
        'GPT-Neo': 'http://39.98.127.31:8012/z',
        'RPM-G2': 'http://121.89.205.93:8012/z'
      },
      prompt: '--你好\n' +
        '-你最喜欢的东西是什么?\n' +
        '-我喜欢优美的环境，安静的庭院，空灵的雨滴，都是令人心悦的。\n' +
        '-你最害怕的东西是什么?\n' +
        '-我害怕人类。',
      activeNames: ['1'],
      engine: engine,
      number: 1,
      response_l: 10,
      top_p: 0,
      temperature: 0.01,
      tableData: [{
        lines: '张三...'
      },
      {
        lines: '李四\n...'
      }]
    }
  },
  methods: {

    handleChange (val) {
      this.$message(val)
    },
    handleClick () {
      this.$message('click on item ' + this.engine)
    },
    handleCommand (command) {
      this.$message('click on item ' + command)
      switch (command) {
        case 'a': this.engine = 'CPM'; break
        case 'b': this.engine = 'RPM'; break
        case 'c': this.engine = 'GPT-Neo'; break
        case 'd': this.engine = 'RPM-G2'; break
      }
    },
    request () {
      let loadingInstance = Loading.service({'text': '请求中..'})
      let data = {
        'prompt': this.prompt,
        'number': this.number,
        'length': this.response_l,
        'top_p': this.top_p,
        'temperature': this.temperature
      }
      this.$message('click on item ' + JSON.stringify(data))
      axios({
        url: this.backends[this.engine],
        method: 'post',
        data
      }).then(res => {
        console.log(
          '-----------------CPM返回数据-------------------',
          res
        )
        var result = res.data.result.map(i => {
          var pos = [i.lastIndexOf('。'), i.lastIndexOf('？'), i.lastIndexOf('！')]
          var maxPos = Math.max(...pos)
          if (maxPos === -1) {
            maxPos = i.length
          }
          return {'lines': i.substring(0, maxPos + 1)}
        })
        this.tableData = result
      })
        .catch(function (error) {
          console.log(error)
          this.tableData = []
        })
        .then(function () {
          loadingInstance.close()
        })
    }
  }
}
</script>

<style scoped>
.el-row {
  margin-bottom: 20px;

  &:last-child {
    margin-bottom: 0;
  }
}

.el-col {
  border-radius: 4px;
}

.bg-purple-dark {
  background: #99a9bf;
}

.bg-purple {
  background: #d3dce6;
}

.bg-purple-light {
  background: #e5e9f2;
}

.grid-content {
  border-radius: 4px;
  min-height: 36px;
}

.row-bg {
  padding: 10px 0;
  background-color: #f9fafc;
}
</style>
