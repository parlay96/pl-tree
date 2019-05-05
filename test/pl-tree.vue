<!--pl-tree（无限极分类）参数如下：
      Attributes（属性）
           * datas： tree树形数据 datas基本数据结构查看： props配置选项
           * props: 配置选项，具体看props配置列表
           * node-key: 每个树节点用来作为唯一标识的属性，整棵树应该是唯一的String,（必须配置）
           accordion：是否每次只打开一个同级树节点展开
           show-checkbox: 节点是否可被选择 默认为false
           select-type: string类型： radio || multiSelect  单选(radio)  多选（multiSelect） // 默认为单选
           show-add: 是否显示可添加节点按钮 默认为false
           show-delete: 是否显示删除节点按钮 默认为false
           show-amend: 是否显示修改节点名称按钮 默认为false
           lable-color="rgb(85, 85, 85)" // 树节点lable字体颜色 // 默认 为#555
           lable-check-color="red" // 树节点被选中状态下的lable字体颜色 // 默认 为无
           lable-font-size="0.98rem" // 树节点lable字体大小 // 默认 1.2rem
      props配置列表  {    lable: 'lable',children: 'children', disabled: boolean  } // 默认值
          lable: 指定节点标签为节点对象的某个属性值-string
          children: 指定子树为节点对象的某个属性值-string
          disabled: 指定节点选择框是否禁用为节点对象的某个属性值-boolean
      methods（方法）
           // 获取当前被勾选的节点 （若节点可被选择（即 show-checkbox 为 true），则返回目前被选中的节点所组成的数组）
           this.$refs.tree.getChecked()
           // 清除当前被勾选的节点 （若节点可被选择（即 show-checkbox 为 true），则清除当前被勾选的节点）
           this.$refs.tree.resetChecked()
           // 调用封装树的过滤方法进行过滤数据
           this.$refs.tree.filter(val) val为输入框的值
           // 调用封装树的设置某个节点的勾选状态方法 （ 通过 setChecked 设置目前勾选的节点，使用此方法必须设置 node-key 属性）
           this.$refs.tree.setChecked(9, true) 9为当前节点对象的node-key值, true选中状态|| false 不选中
           // 强制重新渲染组件方法 (当主动修改值或者其他非响应属性时需要调用该方法)
           this.$refs.tree.renders()
           // 添加节点方法
           this.$refs.tree.addNode(params)
                参数: params = {
                       lable: '' // 属性名必须跟props的lable属性名  保持一致
                       children: '' // 属性名必须跟props的children属性名  保持一致
                       id: '' // 节点的唯一标识（属性名必须与nodeKey属性值） 保持一致
                    }
            // 删除节点方法
            this.$refs.tree.deleteNode(id) // 参数为要删除节点的id
            // 修改节点名称
            this.$refs.tree.amendNode(params)
               参数: params = {
                      value: '' // 要修改节点名称的值
                      id: '' // 参数为要修改节点的id
                    }
      Events（事件）
           @check-change： 选中项的回调
           @node-click: 节点被点击时的回调
           @check-codes-click: 节点被选中的回调
           @append-click: 节点添加按钮的点击回调
           @delete-click: 节点删除按钮的点击回调
           @amend-click: 节点修改按钮的点击回调
       -->
<template>
  <div class="pl-Tree">
    <ul class="pl-menu boxSz" v-if="data.length > 0 && renderComponent">
      <!-- 递归组件-->
      <pl-tree-item v-for="(item, index) in data"
                    :props="defaultProps"
                    :key="index"
                    :show-checkbox="showCheckbox"
                    :show-add="showAdd"
                    :show-delete="showDelete"
                    :show-amend="showAmend"
                    :select-type="selectType"
                    :lable-color="lableColor"
                    :lable-font-size="lableFontSize"
                    :lable-check-color="lableCheckColor"
                    :items="item"/>
    </ul>
    <div v-else class="noData"><div v-if="renderComponent">无相关数据</div></div>
  </div>
</template>

<script>
  import plTreeItem from './pl-tree-item'
  import Bus from '../bus'
  // Tree 树形控件
  export default {
    components: { plTreeItem },
    data () {
      return {
        defaultProps: {
          children: 'children',
          lable: 'lable',
          disabled: true
        },
        nodeItem: {}, // 当前删除，新增，编辑的节点对象
        renderComponent: true, // 刷新组件
        toggleOpenState: false,
        childrenOpen: false, // 为true代表当前defaultProps children的属性名被修改了
        lableOpen: false, // 为true代表当前defaultProps lable的属性名被修改了
        data: [], // 原始数据
        newData: [], // 拷贝原始数据
        values: '' // 保存输入框的值
      }
    },
    watch: {
      datas: {
        handler (newValue) {
          this.initData()
        },
        deep: true
      }
    },
    props: {
      datas: { // 树形tree数据
        type: Array,
        default: () => { return [] }
      },
      nodeKey: {
        type: String,
        default: '$$index'
      },
      defaultExpandedKeys: { // 默认展开的节点的node-key的数组
        type: Array,
        default: () => { return [] }
      },
      props: {
        type: Object,
        default: () => { return {} }
      },
      // 是否每次只打开一个同级树节点展开
      accordion: {
        type: Boolean,
        default: false
      },
      // 节点是否可被选择
      showCheckbox: {
        type: Boolean,
        default: false
      },
      // 选择类型 radio || multiSelect  单选(radio)  多选（multiSelect） // 默认为单选
      selectType: {
        type: String,
        default: 'radio'
      },
      // 是否显示可以添加节点按钮
      showAdd: {
        type: Boolean,
        default: false
      },
      // 是否显示删除节点按钮
      showDelete: {
        type: Boolean,
        default: false
      },
      // 是否显示修改节点名称按钮
      showAmend: {
        type: Boolean,
        default: false
      },
      // lable的颜色
      lableColor: {
        type: String,
        default: '#555'
      },
      // lable的字体大小
      lableFontSize: {
        type: String,
        default: '1.2rem'
      },
      // 节点被选中状态下的lable的字体颜色
      lableCheckColor: {
        type: String,
        default: ''
      }
    },
    created () {
      this.defaultProps = this.extend(this.props, this.defaultProps)
    },
    mounted () {
      this.initData() // 初始化方法
      // 注入事件
      Bus.$on('amendClick', value => { this.amendClicks(value) })
      Bus.$on('appendClick', value => { this.appendClick(value) })
      Bus.$on('deleteClick', value => { this.deleteClick(value) })
      Bus.$on('getCurrentNode', value => { this.getCurrentNodes(value) })
      Bus.$on('nodeClick', value => { this.nodeClick(value) })
    },
    methods: {
      /* 节点key配置参数 */
      extend (userOption, defaultOption) {
        if (!userOption) return defaultOption
        if (userOption.children && defaultOption.children !== userOption.children) {
          this.childrenOpen = true
        }
        if (userOption.lable && defaultOption.lable !== userOption.lable) {
          this.lableOpen = true
        }
        for (var key in defaultOption) {
          if (userOption[key] == null) {
            userOption[key] = defaultOption[key]
          } else if (typeof userOption[key] === 'object') {
            this.extend(userOption[key], defaultOption[key]) // 深度匹配
          }
        }
        return userOption
      },
      // 初始化方法（处理数据）
      initData () {
        // 递归函数处理数据操作
        let classificationFiltra = (params) => {
          let hasChildren = (obj) => obj[this.defaultProps.children] !== undefined &&
            obj[this.defaultProps.children] !== null &&
            obj[this.defaultProps.children].constructor === Array
          // 添加层级（hierarchy）
          let levelsFn = (arr, levels) => {
            levels++
            arr.forEach(obj => {
              obj['hierarchy'] = levels
              if (hasChildren(obj)) {
                levelsFn(obj[this.defaultProps.children], levels)
              }
            })
          }
          levelsFn(params, 0)
          // 添加节点唯一标识（$$index）
          let idx = 0
          let _addIndex = (arr) => {
            arr.forEach(obj => {
              obj['$$index'] = idx++
              if (hasChildren(obj)) {
                obj['toggleOpen'] = false // 默认收起节点开关
                // 设置默认展开节点
                if (this.defaultExpandedKeys.constructor === Array) {
                  this.defaultExpandedKeys.forEach(item => {
                    if (item === obj[this.nodeKey]) {
                      obj['toggleOpen'] = true // 展开节点开关
                    }
                  })
                }
                _addIndex(obj[this.defaultProps.children])
              }
            })
          }
          _addIndex(params)
          // 拷贝
          let _copy = (arr) => arr.forEach(obj => {
            if (this.lableOpen) {
              obj['lable'] = obj[this.defaultProps.lable]
            }
            if (this.childrenOpen) {
              obj['children'] = obj[this.defaultProps.children]
            }
            if (hasChildren(obj)) {
              _copy(obj[this.defaultProps.children])
            }
          })
          _copy(params)
        }
        // 调用递归函数处理节点操作方法
        this.data = JSON.parse(JSON.stringify(this.datas))
        classificationFiltra(this.data)
        // 拷贝原始数据
        this.newData = JSON.parse(JSON.stringify(this.data))
        // 调用当前选中项
        if (this.data.length > 0) {
          this.pitchOn()
        }
      },
      // 修改事件
      amendClicks (item) {
        this.nodeItem = JSON.parse(JSON.stringify(item))
        this.$emit('amend-click', this.qrganizeData(item))
      },
      // 添加按钮事件
      appendClick (item) {
        this.nodeItem = JSON.parse(JSON.stringify(item))
        this.$emit('append-click', this.qrganizeData(item))
      },
      // 删除按钮事件
      deleteClick (item) {
        this.nodeItem = JSON.parse(JSON.stringify(item))
        this.$emit('delete-click', this.qrganizeData(item))
      },
      // 递归过滤函数(筛选)
      filter (val) {
        this.values = val
        if (!val) {
          this.data = JSON.parse(JSON.stringify(this.newData))
        } else {
          // 判断这条分支上有没有匹配到的
          let isNeedBranch = (item, keyword) => {
            let flag1 = false; let flag2 = false
            if (item.lable.indexOf(keyword) > -1) {
              flag1 = true
            } else if (item.children && item.children.length) {
              item.children.forEach(child => {
                if (isNeedBranch(child, keyword)) {
                  flag2 = true
                }
              })
            }
            return flag1 || flag2
          }
          // 给每个分支上有所匹配的设置isNeed
          let setFlag = (data, keyword) => {
            return data.map(item => {
              item.isNeed = isNeedBranch(item, keyword)
              if (item.children && item.children.length) {
                if (this.toggleOpenState) {
                  item.toggleOpen = true // 展开所有子节点
                }
                setFlag(item.children, keyword)
              }
              return item
            })
          }
          // 过滤掉分支上isNeed为false的元素
          let treeFilter = (data) => {
            return data.filter((item, index) => {
              if (item && item.children && item.children.length) {
                item.children = treeFilter(item.children)
              }
              return item.isNeed
            })
          }
          // 拷贝数据
          let data = JSON.parse(JSON.stringify(this.newData))
          // 调用过滤方法
          let screenData = treeFilter(setFlag(data, val))
          // 赋值数据给视图
          this.data = JSON.parse(JSON.stringify(screenData))
        }
      },
      // 递归查询函数(模糊查询)
      queryFiltra (params, type, val) {
        let DataSource = [] // 当前选中项
        let hierarchy = '' // 层级
        let hasChildren = (obj) => obj.children !== undefined && obj.children !== null
        let recursionFiltra = (arr) => arr.forEach(obj => {
          hierarchy = params.hierarchy || -1
          if (params.$$index === obj.$$index) {
            if (type === '展开收起') {
              obj.toggleOpen = !params.toggleOpen // 设置状态
            } else if (type === '点击勾选') {
              obj.checked = !params.checked // 设置选中状态 取反
            }
          }
          // 如果是单选
          if (this.selectType === 'radio' && type === '点击勾选') {
            // 除了当前点击的节点选中，其余节点全部设置为未选中
            if (params.$$index !== obj.$$index && this.selectType === 'radio') {
              obj.checked = false
            }
          }
          // 每次只打开一个同级树节点展开
          if (this.accordion && hierarchy && type === '展开收起') {
            // 除了当前点击的节点展开，其余同级节点全部收起
            if (params.$$index !== obj.$$index && hierarchy === obj.hierarchy) {
              obj.toggleOpen = false // 设置状态
            }
          }
          if (type === '设置勾选状态') {
            if (obj[this.nodeKey] !== undefined) {
              if (params.toString() === obj[this.nodeKey].toString()) {
                obj.checked = val // 设置状态
              }
            } else {
              console.assert(false, '你绑定的node-key值，在节点在对象中找不到，所以无法使用setChecked方法勾选节点')
            }
          }
          if (type === '当前选中项') {
            if (obj.checked) {
              let objData = this.qrganizeData(obj)
              DataSource.push(objData)
            }
          }
          if (type === '清除当前被勾选的节点') {
            obj.checked = false
          }
          if (hasChildren(obj)) {
            recursionFiltra(obj.children)
          }
        })
        this.toggleOpenState = true
        if (type === '展开收起') {
          this.toggleOpenState = false
        }
        // 执行递归查询方法
        recursionFiltra(this.newData)
        // 调用筛选函数
        this.filter(this.values)
        // 返回当前选中项
        if (type === '当前选中项') {
          return DataSource
        }
      },
      // 当前选中项方法
      pitchOn () {
        // 获取当前选中项
        let selectedData = this.queryFiltra({}, '当前选中项')
        // 暴露当前选中项
        this.$emit('check-change', selectedData)
      },
      // 节点被点击事件
      nodeClick (item) {
        // 展开收起按钮 代表有层级
        if (item.children !== undefined && item.children !== null) {
          // 调用递归查询函数
          this.queryFiltra(item, '展开收起')
        }
        // 节点被点击时的回调
        this.$emit('node-click', this.qrganizeData(item))
      },
      // 点击勾选按钮
      async getCurrentNodes (item) {
        if (!item.checked) {
          this.$emit('check-codes-click', this.qrganizeData(item))
        }
        // 调用递归查询函数
        await this.queryFiltra(item, '点击勾选')
        // 获取当前选中项
        await this.pitchOn()
      },
      // 获取当前被勾选的节点
      getChecked () {
        // 节点可被选择的情况下才能去获取勾选的节点
        if (this.showCheckbox) {
          // 获取当前选中项
          let selectedData = this.queryFiltra({}, '当前选中项')
          return selectedData
        }
      },
      // 清除当前被勾选的节点
      resetChecked () {
        // 节点可被选择的情况下才能去清除当前被勾选的节点
        if (this.showCheckbox) {
          // 清除当前被勾选的节点
          this.queryFiltra({}, '清除当前被勾选的节点')
        }
      },
      // 设置某个节点的勾选状态
      async setChecked (id, val) {
        if (id && val) {
          // 设置某个节点的勾选状态
          await this.queryFiltra(id, '设置勾选状态', val)
          // 获取勾选数据
          await this.pitchOn()
        } else {
          console.assert(false, '无节点id或者值， 所以无法使用setChecked方法勾选节点')
        }
      },
      // 添加节点
      addNode (params) {
        if (Object.prototype.toString.call(params) === '[object Object]') {
          if (params[this.defaultProps.lable] && params[this.nodeKey]) {
            let obj = JSON.parse(JSON.stringify(params))
            obj.children = obj.children || []
            obj['$$index'] = obj.id
            obj.hierarchy = this.nodeItem.hierarchy + 1 // 楼层(父级楼层加1)
            this.recursive(obj, '添加节点')
          } else {
            console.assert(false, '添加节点的对象属性名必须与props配置项一致，详细请看文档配置')
          }
        } else {
          console.assert(false, '添加节点的参数必须是个对象，详细请看文档配置')
        }
      },
      // 删除节点
      deleteNode (id) {
        if (id || id.toString() === '0') {
          this.recursive({ id: id }, '删除节点')
        } else {
          console.assert(false, '删除节点的参数id必须存在')
        }
      },
      // 修改节点名称
      amendNode (params) {
        if (params.id && params.value) {
          this.recursive(params, '修改节点')
        } else {
          console.assert(false, '修改节点的参数有误，请看文档')
        }
      },
      // 组装数据方法
      qrganizeData (item) {
        let frlit = (params) => {
          for (let i in params) {
            if (this.childrenOpen) {
              delete params[i]['children']
            }
            if (this.lableOpen) {
              delete params[i]['lable']
            }
            if (this.nodeKey !== '$$index') {
              delete params[i]['$$index']
            }
            delete params[i]['toggleOpen']
            delete params[i]['hierarchy']
            delete params[i]['checked']
            if (params[i].hasOwnProperty(this.defaultProps.children)) {
              frlit(params[i][this.defaultProps.children])
            }
          }
          return params
        }
        let datas = frlit([JSON.parse(JSON.stringify(item))])
        return datas[0]
      },
      // 递归数据方法(增删改)
      recursive (newObj, type) {
        let hasChildren = (obj) => obj[this.defaultProps.children] !== undefined &&
          obj[this.defaultProps.children] !== null &&
          obj[this.defaultProps.children].constructor === Array
        let fals = false
        let exclude = (arr) => {
          arr.forEach((objs, index) => {
            if (objs[this.nodeKey].toString() === this.nodeItem.id.toString() && type === '添加节点') {
              if (objs.children.constructor === Array) {
                objs.children.push(newObj)
              }
            }
            if (objs[this.nodeKey].toString() === newObj.id.toString() && type === '删除节点') {
              fals = true
              arr.splice(index, 1)
            }
            if (objs[this.nodeKey].toString() === newObj.id.toString() && type === '修改节点') {
              fals = true
              objs[this.defaultProps.lable] = newObj.value
            }
            if (hasChildren(objs)) {
              exclude(objs.children)
            }
          })
        }
        exclude(this.newData)
        if (!fals && type === '删除节点') {
          console.assert(false, '删除节点的失败, id不存在')
        }
        if (!fals && type === '修改节点') {
          console.assert(false, '修改节点的失败, id不存在')
        }
        this.data = JSON.parse(JSON.stringify(this.newData))
      },
      // 强制刷新组件
      renders () {
        this.renderComponent = false
        this.$nextTick(() => {
          this.renderComponent = true
        })
      }
    },
    beforeDestroy () {
      // 清除事件
      Bus.$off('getCurrentNode')
      Bus.$off('nodeClick')
      Bus.$off('appendClick')
      Bus.$off('deleteClick')
      Bus.$off('amendClick')
    }
  }
</script>

<style scoped lang="less">
  .pl-Tree {width: 100%;height: 100%;box-sizing: border-box;overflow-y: auto;
    .boxSz {box-sizing: border-box;}
    .beyond {overflow: hidden;text-overflow:ellipsis;white-space: nowrap;}
    .pl-menu {list-style: none;position: relative;margin: 0;padding: 0;}
    .noData{width: 100%;text-align: center;font-size: 1.1rem;padding-top: 1rem;color: #999;}
  }
</style>
