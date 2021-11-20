## element table列数不定

在项目里有时会遇到渲染的table列数是不定的，后端可能今天传10列给你，后天可能就传20列给你了。遇到这种情况，我们就不能用element table官网上写的那种方式了。而是，我们需要使用数组的方式来进行遍历。

1.html结构

```js
 <el-table
      :data="tableData.list"
      style="width: 100%"
      :height="500"
    >
      <el-table-column
        v-for="(column, index) in tableData.headers"
        :key="index"
        :fixed="column === '序号' ? 'left' : false"
        :label="column"
        width="120"
      >
        <template slot-scope="scope">
          <template v-if="scope.row[column].error">
            <el-popover trigger="hover" placement="top">
              <p>{{ scope.row[column].tooltip }}</p>
              <div slot="reference" class="name-wrapper">
                <span class="error-content pd4">
                  {{ scope.row[column].content || "错误信息" }}
                </span>
              </div>
            </el-popover>
          </template>
          <template v-else>
            <el-popover
              trigger="hover"
              placement="top"
              v-if="scope.row[column].tooltip"
            >
              <p>{{ scope.row[column].tooltip }}</p>
              <div slot="reference" class="name-wrapper">
                <div class="normal-content">
                  {{ scope.row[column].content || "成功" }}
                </div>
              </div>
            </el-popover>
            <div v-else class="normal-content">
              {{ scope.row[column].content || "成功" }}
            </div>
          </template>
        </template>
      </el-table-column>
    </el-table>
```

其中的重点就是对el-table-column对tableData.headers的遍历，因为是遍历出来的，所以不管tableData.headers里的元素有多少个，table就能显示多少列

2.数据的处理

```js
tableData: {
        list: [],
        header: []
      },
```

tableData.header就是需要遍历出来的表头，tableData.list就是具体的每一项了

tableData.header的结构：

```
[
    "序号",
    "CARD_CODE",
    "COMPANY_ID",
    "COORDINATE_SYSTEM",
    "DEVICENUM",
    "DEVICE_MAC",
    "END_TIME",
    "HISTORY_SSID",
    "IMEI",
    "IMSI",
    "LATITUDE",
    "LONGITUDE",
    "MAC",
    "MODEL",
    "NAME",
    "OS_VERSION",
    "PHONE",
    "POWER",
    "SERVICECODE",
    "SESSION_ID",
    "START_TIME",
    "STATION",
    "S_MAC",
    "S_SSID"
]
```

tableData.list的结构：

```js
[
    {
    "序号": {
        "content": 1,
        "error": false,
        "tooltip": "/opt/filecenter/ftp/autotest/szgd/DWZM/20211111/15/20210727164547_559_440305_980609862_07.log"
    },
    "CARD_CODE": {
        "content": "238108406227136980",
        "error": false
    },
    "COMPANY_ID": {
        "content": "63897199358984",
        "error": false
    },
    "COORDINATE_SYSTEM": {
        "content": "04",
        "error": false
    },
    "DEVICENUM": {
        "content": "11764368047",
        "error": false
    },
    "DEVICE_MAC": {
        "content": "47-67-3D-7A-76-2B",
        "error": false
    },
    "END_TIME": {
        "content": "1636616178",
        "error": false
    },
    "HISTORY_SSID": {
        "content": "awyuut,iphqhgij,afqmlqzw,ohnw",
        "error": false
    },
    "IMEI": {
        "content": "63791993674",
        "error": false
    },
    "IMSI": {
        "content": "47284305522",
        "error": false
    },
    "LATITUDE": {
        "content": "23.096117",
        "error": false
    },
    "LONGITUDE": {
        "content": "112.488113",
        "error": false
    },
    "MAC": {
        "content": "41-19-16-17-1A-65",
        "error": false
    },
    "MODEL": {
        "content": "baiJAIHlX",
        "error": false
    },
    "NAME": {
        "content": "test_man",
        "error": false
    },
    "OS_VERSION": {
        "content": "YLPzb",
        "error": false
    },
    "PHONE": {
        "content": "13612352012",
        "error": false
    },
    "POWER": {
        "content": "-41",
        "error": false
    },
    "SERVICECODE": {
        "content": "74609542151040",
        "error": false
    },
    "SESSION_ID": {
        "content": "DE2CE722B024404F84833FCC637E8390",
        "error": false
    },
    "START_TIME": {
        "content": "1636616178",
        "error": false
    },
    "STATION": {
        "content": "test-1636616178",
        "error": false
    },
    "S_MAC": {
        "content": "65-15-25-2E-75-18",
        "error": false
    },
    "S_SSID": {
        "content": "fcgz,dmglk",
        "error": false
    },
    "file": {
        "content": "/opt/filecenter/ftp/autotest/szgd/DWZM/20211111/15/20210727164547_559_440305_980609862_07.log",
        "error": false
    }
	}
]
```

其中tableData.list是请求回来的数据，前端进行了一次格式化,格式化的使用是使用的forEach对每一项做的处理，所以不用关心后台传回来有几项，格式化后使得其变成这种格式。

```js
created () {
    this.$axios.get('/user/tableData').then((res) => {
      this.tableData.list = [...res.data.data.map((column,index) => this.formatDataHeader(column,index))]
      console.log(this.tableData.list);
    })
  },
```

tableData.header的处理，则是使用了watch来监听：

```js
watch: {
    'tableData.list' (newVal) {
      const headerSet = new Set()
      newVal?.forEach(column => Object.keys(column).map(item => (item !== 'file') && headerSet.add(item)))
      this.tableData.headers = [...headerSet]
      // console.log(this.tableData.headers);
    },
  },
```

完整的代码，可以看我的demo，这边只是给出了一些思路，如有不对的，大家可以纠正。

代码：https://github.com/rui-rui-an/dynamictable
