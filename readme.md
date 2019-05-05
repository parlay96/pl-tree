# pl-tree
> 一个基于vue 的树组件
# method of use
> npm i pl-tree
# Component API
  https://github.com/penglei1996/pl-tree/wiki/Component-API
# Attributes（属性）
        datas： tree树形数据 datas基本数据结构请查看： props配置选项

        props: 配置选项，具体看props配置列表

        node-key: 每个树节点用来作为唯一标识的属性，整棵树应该是唯一的String,（必须配置）

        default-expanded-keys: 默认展开的节点的node-key的数组	array 需要注意的是 ，
                            此时必须设置node-key, default-expanded-keys格式如 [1， 2. 3]

        accordion：是否每次只打开一个同级树节点展开

        show-checkbox: 节点是否可被选择 默认为false

        select-type: string类型： radio || multiSelect  单选(radio)  多选（multiSelect） // 默认为单选

        show-add: 是否显示可添加节点按钮 默认为false

        show-delete: 是否显示删除节点按钮 默认为false

        show-amend: 是否显示修改节点名称按钮 默认为false

        lable-color="rgb(85, 85, 85)" // 树节点lable字体颜色 // 默认 为#555

        lable-check-color="red" // 树节点被选中状态下的lable字体颜色 // 默认 为无

        lable-font-size="0.98rem" // 树节点lable字体大小 // 默认 1.2rem

# pl-tree-item-node （属性）
        disabled: 指定节点选择框是否禁用  节点属性值必须为props配置里面disabled属性的值 否则无效

        nodeShowAdd: 指定节点是否显示可添加节点按钮-boolean true不显示， 其他值都为显示

        nodeShowDelete: 指定节点是否显示可删除节点按钮-boolean true不显示， 其他值都为显示

        nodeShowAmend: 指定节点是否显示可修改节点按钮-boolean true不显示， 其他值均为显示

# props配置
        { lable: 'lable',children: 'children', disabled: boolean  } // 默认值

        lable: 指定节点标签为节点对象的某个属性值-string

        children: 指定子树为节点对象的某个属性值-string

        disabled: 指定节点选择框是否禁用为节点对象的某个属性值-boolean 默认值为 true

# methods（方法）
       this.$refs.tree.getChecked()  --> 获取当前被勾选的节点 （若节点可被选择
      （即 show-checkbox 为 true）， 则返回目前被选中的节 点所组成的数组）

       this.$refs.tree.resetChecked() --> 清除当前被勾选的节点 （若节点可被选择
      （即 show-checkbox 为 true），则清除当前被勾选的 节点）

       this.$refs.tree.filter(val)  -->调用封装树的过滤方法进行过滤数据 val为输入
       框的值

       this.$refs.tree.setChecked(9, true) --> 调用封装树的设置某个节点的勾选状态方法
       9为当前节点对象的node-key值, true 选中状态|| false 不选中 --> （ 通过
       setChecked 设置目前勾选的节点，使用此方法必须设置 node-key 属性）

       this.$refs.tree.renders() --> 强制重新渲染组件方法 (当主动修改值或者
       其他非响应属性时需要调用该方法)

       this.$refs.tree.addNode(params) --> 添加节点方法
          参数: params = {
                    lable: '' // 属性名必须跟props的lable属性名  保持一致
                    children: '' // 属性名必须跟props的children属性名  保持一致
                     id: '' // 节点的唯一标识（属性名必须与nodeKey属性值） 保持一致
             }

       this.$refs.tree.deleteNode(id) --> 删除节点方法, 参数为要删除节点的id

       this.$refs.tree.amendNode(params) --> 修改节点名称
           参数: params = {
                   value: '' // 要修改节点名称的值
                   id: '' // 参数为要修改节点的id
                 }

# Events（事件）
      @check-change： 选中项的回调

      @node-click: 节点被点击时的回调

      @check-codes-click: 节点被选中的回调

      @append-click: 节点添加按钮的点击回调

      @delete-click: 节点删除按钮的点击回调

      @amend-click: 节点修改按钮的点击回调
