<template>
  <div class="search-form">
    <!-- 头部tab切换 -->
    <el-row type="flex" class="search-tab">
      <span
        v-for="(item, index) in tabs"
        :key="index"
        @click="handleSearchTab(item, index)"
        :class="{active: index === currentTab}"
      >
        <i :class="item.icon"></i>
        {{item.name}}
      </span>
    </el-row>

    <el-form class="search-form-content" ref="form" label-width="80px">
      <el-form-item label="出发城市">
        <!-- fetch-suggestions 返回输入建议的方法 -->
        <!-- select 点击选中建议项时触发 -->
        <el-autocomplete
          v-model="form.departCity"
          :fetch-suggestions="queryDepartSearch"
          :trigger-on-focus="false"
          :highlight-first-item="true"
          placeholder="请搜索出发城市"
          @select="handleDepartSelect"
          class="el-autocomplete"
        ></el-autocomplete>
      </el-form-item>
      <el-form-item label="到达城市">
        <el-autocomplete
          v-model="form.destCity"
          :fetch-suggestions="queryDestSearch"
          :trigger-on-focus="false"
          :highlight-first-item="true"
          placeholder="请搜索到达城市"
          @select="handleDestSelect"
          class="el-autocomplete"
        ></el-autocomplete>
      </el-form-item>
      <el-form-item label="出发时间">
        <!-- change 用户确认选择日期时触发 -->
        <el-date-picker
          type="date"
          v-model="form.departDate"
          placeholder="请选择日期"
          style="width: 100%;"
          @change="handleDate"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label>
        <el-button style="width:100%;" type="primary" icon="el-icon-search" @click="handleSubmit">搜索</el-button>
      </el-form-item>
      <div class="reverse">
        <span @click="handleReverse">换</span>
      </div>
    </el-form>
  </div>
</template>

<script>
import moment from 'moment'

export default {
  data() {
    return {
      tabs: [
        { icon: 'iconfont icondancheng', name: '单程' },
        { icon: 'iconfont iconshuangxiang', name: '往返' }
      ],
      currentTab: 0,
      form: {
        departCity: '',
        departCode: '',
        destCity: '',
        destCode: '',
        departDate: ''
      }
    }
  },
  methods: {
    // tab切换时触发
    handleSearchTab(item, index) {
      if (index === 1) {
        this.$confirm('目前暂不支持往返，请使用单程选票！', '提示', {
          confirmButtonText: '确定',
          showCancelButton: false,
          type: 'warning'
        })
      }
    },

    // 每个函数应该做好, 并且只做一件事情
    getCityList(value) {
      return this.$axios({
        url: '/airs/city',
        params: {
          name: value
        }
      }).then(res => {
        console.log(res.data)
        // 我们拿到了res.data.data
        // 但是里面没有 value 值 而是 name 属性
        const suggestions = res.data.data.map(city => {
          return {
            // 城市名的最后如果是市字,要删掉
            value: city.name.replace(/市$/, ''),
            code: city.sort
          }
        })
        // 过滤掉所有不带有 code 的城市
        return suggestions.filter(city => {
          return city.code
        })
      })
    },

    // 出发城市输入框获得焦点时触发
    // value 是选中的值，cb是回调函数，接收要展示的列表
    queryDepartSearch(value, showList) {
      // 我调用封装好的请求函数, 应该在里面能够获取到数据,
      // 拿到了这个数据进行下一步 将传回来的数据执行 showList
      this.getCityList(value).then(suggestions => {
        // 这里获取到列表, 就应该先将列表的第一项设为默认 code
        if (suggestions.length > 0) {
          this.form.departCode = suggestions[0].code
          // 显示推荐列表
          showList(suggestions)
        } else {
          showList([
            {
              value: '没有找到该城市'
            }
          ])
        }
      })
    },

    // 目标城市输入框获得焦点时触发
    // value 是选中的值，showList是回调函数，接收要展示的列表
    queryDestSearch(value, showList) {
      this.getCityList(value).then(suggestions => {
        if (suggestions.length > 0) {
          this.form.destCode = suggestions[0].code
          showList(suggestions)
        } else {
          showList([
            {
              value: '没有找到该城市'
            }
          ])
        }
      })
    },

    // 出发城市下拉选择时触发
    handleDepartSelect(item) {
      console.log(11111111)
      // 建议被选择后,会带上一个被选项 item 对象
      // 将里面的 code 存放到 表单的出发城市代码
      console.log(item)
      this.form.departCode = item.code
    },

    // 目标城市下拉选择时触发
    handleDestSelect(item) {
      console.log(item)
      this.form.destCode = item.code
    },

    // 确认选择日期时触发
    handleDate(value) {
      // 这里是日期选中的的事件触发函数
      // 接收到一个参数, 就是选中的那个时间对象
      // 需要利用 momentjs 转换成合适的格式
      // value 就是时间对象
      // 原生日期对象本来没有格式化函数, 所以要转换成 moment 的对象格式
      this.form.departDate = moment(value).format('YYYY-MM-DD')
    },

    // 触发和目标城市切换时触发
    handleReverse() {
      const oldDestCity = this.form.destCity
      const oldDestCode = this.form.destCode

      this.form.destCity = this.form.departCity
      this.form.destCode = this.form.departCode

      this.form.departCity = oldDestCity
      this.form.departCode = oldDestCode
    },

    // 提交表单是触发
    handleSubmit() {
      console.log(this.form)
      // 现在不仅仅发起跳转, 还要把搜索内容记录在 vuex 上面
      // 1. 提取参数对象
      const query = {
        departCity: this.form.departCity.replace(/市$/, ''),
        departCode: this.form.departCode,
        destCity: this.form.destCity.replace(/市$/, ''),
        destCode: this.form.destCode,
        departDate: this.form.departDate
      }
      // 2.讲这个搜索参数对象存放进 vuex 历史记录当中
      // 就要修改 state
      this.$store.commit('history/addHistory', query)
      // 利用编程式导航跳转页面
      this.$router.push({
        path: '/air/flights',
        // query: this.form
        query
      })
    }
  },
  mounted() {}
}
</script>

<style scoped lang="less">
.search-form {
  border: 1px #ddd solid;
  border-top: none;
  width: 360px;
  height: 350px;
  box-sizing: border-box;
}

.search-tab {
  span {
    display: block;
    flex: 1;
    text-align: center;
    height: 48px;
    line-height: 42px;
    box-sizing: border-box;
    border-top: 3px #eee solid;
    background: #eee;
  }

  .active {
    border-top-color: orange;
    background: #fff;
  }

  i {
    margin-right: 5px;
    font-size: 18px;

    &:first-child {
      font-size: 16px;
    }
  }
}

.search-form-content {
  padding: 15px 50px 15px 15px;
  position: relative;

  .el-autocomplete {
    width: 100%;
  }
}

.reverse {
  position: absolute;
  top: 35px;
  right: 15px;

  &:after,
  &:before {
    display: block;
    content: '';
    position: absolute;
    left: -35px;
    width: 25px;
    height: 1px;
    background: #ccc;
  }

  &:after {
    top: 0;
  }

  &:before {
    top: 60px;
  }

  span {
    display: block;
    position: absolute;
    top: 20px;
    right: 0;
    font-size: 12px;
    background: #999;
    color: #fff;
    width: 20px;
    height: 20px;
    line-height: 18px;
    text-align: center;
    border-radius: 2px;
    cursor: pointer;

    &:after,
    &:before {
      display: block;
      content: '';
      position: absolute;
      left: 10px;
      width: 1px;
      height: 20px;
      background: #ccc;
    }

    &:after {
      top: -20px;
    }

    &:before {
      top: 20px;
    }
  }
}
</style>