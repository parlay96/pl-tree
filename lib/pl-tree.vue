<template>
  <div class="pl-Tree">
    <ul class="pl-menu boxSz" v-if="data.length > 0 && renderComponent">
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
  export default {
    components: { plTreeItem },
    data () {
      return {
        defaultProps: {
          children: 'children',
          lable: 'lable',
          disabled: true
        },
        nodeItem: {},
        renderComponent: true,
        toggleOpenState: false,
        childrenOpen: false,
        lableOpen: false,
        data: [],
        newData: [],
        values: ''
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
      datas: {
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
      accordion: {
        type: Boolean,
        default: false
      },
      showCheckbox: {
        type: Boolean,
        default: false
      },
      selectType: {
        type: String,
        default: 'radio'
      },
      showAdd: {
        type: Boolean,
        default: false
      },
      showDelete: {
        type: Boolean,
        default: false
      },
      showAmend: {
        type: Boolean,
        default: false
      },
      lableColor: {
        type: String,
        default: '#555'
      },
      lableFontSize: {
        type: String,
        default: '1.2rem'
      },
      lableCheckColor: {
        type: String,
        default: ''
      }
    },
    created () {
      this.defaultProps = this.extend(this.props, this.defaultProps)
    },
    mounted () {
      this.initData()
      Bus.$on('amendClick', value => { this.amendClicks(value) })
      Bus.$on('appendClick', value => { this.appendClick(value) })
      Bus.$on('deleteClick', value => { this.deleteClick(value) })
      Bus.$on('getCurrentNode', value => { this.getCurrentNodes(value) })
      Bus.$on('nodeClick', value => { this.nodeClick(value) })
    },
    methods: {
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
            this.extend(userOption[key], defaultOption[key])
          }
        }
        return userOption
      },
      initData () {
        let classificationFiltra = (params) => {
          let hasChildren = (obj) => obj[this.defaultProps.children] !== undefined &&
            obj[this.defaultProps.children] !== null &&
            obj[this.defaultProps.children].constructor === Array
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
        this.data = JSON.parse(JSON.stringify(this.datas))
        classificationFiltra(this.data)
        this.newData = JSON.parse(JSON.stringify(this.data))
        if (this.data.length > 0) {
          this.pitchOn()
        }
      },
      amendClicks (item) {
        this.nodeItem = JSON.parse(JSON.stringify(item))
        this.$emit('amend-click', this.qrganizeData(item))
      },
      appendClick (item) {
        this.nodeItem = JSON.parse(JSON.stringify(item))
        this.$emit('append-click', this.qrganizeData(item))
      },
      deleteClick (item) {
        this.nodeItem = JSON.parse(JSON.stringify(item))
        this.$emit('delete-click', this.qrganizeData(item))
      },
      filter (val) {
        this.values = val
        if (!val) {
          this.data = JSON.parse(JSON.stringify(this.newData))
        } else {
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
          let setFlag = (data, keyword) => {
            return data.map(item => {
              item.isNeed = isNeedBranch(item, keyword)
              if (item.children && item.children.length) {
                if (this.toggleOpenState) {
                  item.toggleOpen = true
                }
                setFlag(item.children, keyword)
              }
              return item
            })
          }
          let treeFilter = (data) => {
            return data.filter((item, index) => {
              if (item && item.children && item.children.length) {
                item.children = treeFilter(item.children)
              }
              return item.isNeed
            })
          }
          let data = JSON.parse(JSON.stringify(this.newData))
          let screenData = treeFilter(setFlag(data, val))
          this.data = JSON.parse(JSON.stringify(screenData))
        }
      },
      queryFiltra (params, type, val) {
        let DataSource = []
        let hierarchy = ''
        let hasChildren = (obj) => obj.children !== undefined && obj.children !== null
        let recursionFiltra = (arr) => arr.forEach(obj => {
          hierarchy = params.hierarchy || -1
          if (params.$$index === obj.$$index) {
            if (type === '展开收起') {
              obj.toggleOpen = !params.toggleOpen
            } else if (type === '点击勾选') {
              obj.checked = !params.checked
            }
          }
          if (this.selectType === 'radio' && type === '点击勾选') {
            if (params.$$index !== obj.$$index && this.selectType === 'radio') {
              obj.checked = false
            }
          }
          if (this.accordion && hierarchy && type === '展开收起') {
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
        recursionFiltra(this.newData)
        this.filter(this.values)
        if (type === '当前选中项') {
          return DataSource
        }
      },
      pitchOn () {
        let selectedData = this.queryFiltra({}, '当前选中项')
        this.$emit('check-change', selectedData)
      },
      nodeClick (item) {
        if (item.children !== undefined && item.children !== null) {
          // 调用递归查询函数
          this.queryFiltra(item, '展开收起')
        }
        this.$emit('node-click', this.qrganizeData(item))
      },
      async getCurrentNodes (item) {
        if (!item.checked) {
          this.$emit('check-codes-click', this.qrganizeData(item))
        }
        await this.queryFiltra(item, '点击勾选')
        await this.pitchOn()
      },
      getChecked () {
        if (this.showCheckbox) {
          let selectedData = this.queryFiltra({}, '当前选中项')
          return selectedData
        }
      },
      resetChecked () {
        if (this.showCheckbox) {
          this.queryFiltra({}, '清除当前被勾选的节点')
        }
      },
      async setChecked (id, val) {
        if (id && val) {
          await this.queryFiltra(id, '设置勾选状态', val)
          await this.pitchOn()
        } else {
          console.assert(false, '无节点id或者值， 所以无法使用setChecked方法勾选节点')
        }
      },
      addNode (params) {
        if (Object.prototype.toString.call(params) === '[object Object]') {
          if (params[this.defaultProps.lable] && params[this.nodeKey]) {
            let obj = JSON.parse(JSON.stringify(params))
            obj.children = obj.children || []
            obj['$$index'] = obj.id
            obj.hierarchy = this.nodeItem.hierarchy + 1
            this.recursive(obj, '添加节点')
          } else {
            console.assert(false, '添加节点的对象属性名必须与props配置项一致，详细请看文档配置')
          }
        } else {
          console.assert(false, '添加节点的参数必须是个对象，详细请看文档配置')
        }
      },
      deleteNode (id) {
        if (id || id.toString() === '0') {
          this.recursive({ id: id }, '删除节点')
        } else {
          console.assert(false, '删除节点的参数id必须存在')
        }
      },
      amendNode (params) {
        if (params.id && params.value) {
          this.recursive(params, '修改节点')
        } else {
          console.assert(false, '修改节点的参数有误，请看文档')
        }
      },
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
      renders () {
        this.renderComponent = false
        this.$nextTick(() => {
          this.renderComponent = true
        })
      }
    },
    beforeDestroy () {
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
