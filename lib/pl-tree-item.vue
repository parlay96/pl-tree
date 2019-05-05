<template>
  <li class="pl-submenu">
    <!--background: items.hierarchy === 1 ? 'transparent' : 'rgba(230,230,250,'+ (items.hierarchy / 12) + ')'-->
    <div :style="{ paddingLeft: 0.8 * items.hierarchy + 'rem'}"
         class="pl-submenu__title boxSz border-wbc borderBarBottom">
      <div class="sub-title boxSz fl" :style="{width: subTitleWidth}" @click="nodeClicks(items)">
        <p class="unfold fl" v-if="!items.toggleOpen && items.children && items.children.length>0">
          <i class="iconfont icon-jia"/>
        </p>
        <p class="unfold fl" v-if="items.toggleOpen && items.children && items.children.length>0">
          <i class="iconfont icon-jianhao" style="font-size: 1.28rem"/>
        </p>
        <p class="tits boxSz fl" :style="styleObject">
          <span :style="{ color: items.checked && lableCheckColor !== ''? lableCheckColor : styleObject.color }">
            {{ items.lable }}
          </span>
        </p>
      </div>
      <div class="btn-chb" :style="{width: btnChbWidth}">
        <p class="treeItemBtn fr"
           :style="{ width: treeBtnWidth }"
           @click="deletes(items)"
           v-if="showDelete && items.nodeShowDelete !== true">
          <i class="icon-qingkongshanchu iconfont" style="font-size: 1.2rem"/>
        </p>
        <p class="treeItemBtn fr"
           :style="{ width: treeBtnWidth }"
           @click="append(items)"
           v-if="showAdd && items.nodeShowAdd !== true">
          <i class="icon-jiahao iconfont"/>
        </p>
        <p class="treeItemBtn fr"
           @click="amend(items)"
           :style="{ width: treeBtnWidth }"
           v-if="showAmend && items.nodeShowAmend !== true">
          <i class="icon-xiugai iconfont"/>
        </p>
        <p class="treeItemBtn fr"
           @click="getCurrentNode(items)"
           :style="{ width: treeBtnWidth }"
           v-if="showCheckbox">
          <span style="color: #D9D9D9;" v-if="items.disabled === props.disabled">
            <i class="icon-choosehandle iconfont" v-if="selectType === 'radio'"/>
            <i class="icon-duoxuan-yixuan iconfont" style="font-size: 1.45rem" v-else/>
          </span>
          <span style="color: #D9D9D9;" v-else-if="!items.checked">
            <i class="icon-gouxuan-weixuanzhong-xianxingyuankuang iconfont" v-if="selectType === 'radio'"/>
            <i class="icon-kuang iconfont" v-else style="font-size: 1.45rem"/>
          </span>
          <span v-else-if="items.checked" style="color: #1ab399">
            <i class="icon-choosehandle iconfont" v-if="selectType === 'radio'"/>
            <i class="icon-duoxuan-yixuan iconfont" v-else style="font-size: 1.45rem"/>
          </span>
        </p>
      </div>
    </div>
    <ul class="pl-menu boxSz"
        v-if="items.children && items.children.length>0 && items.toggleOpen">
      <pl-tree-item v-for="(submenu,index) in items.children"
                    :props="props"
                    :items="submenu"
                    :show-add="showAdd"
                    :show-delete="showDelete"
                    :show-checkbox="showCheckbox"
                    :select-type="selectType"
                    :show-amend="showAmend"
                    :lable-color="lableColor"
                    :lable-font-size="lableFontSize"
                    :lable-check-color="lableCheckColor"
                    :key="index"/>
    </ul>
  </li>
</template>

<script>
  import Bus from '../bus'
  export default {
    name: 'PlTreeItem',
    props: {
      items: {
        type: Object, default: () => {}
      },
      props: {
        type: Object, default: () => {}
      },
      showCheckbox: {
        type: Boolean
      },
      showAdd: {
        type: Boolean
      },
      showDelete: {
        type: Boolean
      },
      showAmend: {
        type: Boolean
      },
      selectType: {
        type: String,
        default: ''
      },
      lableColor: {
        type: String,
        default: ''
      },
      lableFontSize: {
        type: String,
        default: ''
      },
      lableCheckColor: {
        type: String,
        default: ''
      }
    },
    data () {
      return {
        activesIndex: '',
        styleObject: {},
        btnChbWidth: 'auto', // 按钮区别的总长
        subTitleWidth: 'auto', // tree lable的长度
        treeBtnWidth: '2rem' // 单个按钮的长度
      }
    },
    mounted () {
      // 添加lable样式
      this.styleObject = {
        color: this.lableColor,
        fontSize: this.lableFontSize
      }
      // 计算宽度
      let data = [this.showCheckbox, this.showAdd, this.showDelete, this.showAmend]
      let unit = 0
      data.forEach(item => {
        if (item) unit += parseInt(this.treeBtnWidth)
      })
      this.subTitleWidth = 'calc(100% - ' + (unit + 1) + 'rem)'
      this.btnChbWidth = unit + 'rem'
    },
    methods: {
      // 修改按钮
      amend (item) {
        Bus.$emit('amendClick', item)
      },
      // 点击勾选按钮
      getCurrentNode (item) { if (!item.disabled) Bus.$emit('getCurrentNode', item) },
      // 添加点击事件
      append (item) { Bus.$emit('appendClick', item) },
      // 删除点击事件
      deletes (item) { Bus.$emit('deleteClick', item) },
      // 节点被点击的事件
      nodeClicks (item) {
        this.activesIndex = item.$$index
        Bus.$emit('nodeClick', item)
      }
    }
  }
</script>

<style scoped lang="less">
  .fr{float: right}
  .fl{float: left;}  .boxSz {box-sizing: border-box;}
  .pl-menu {list-style: none;position: relative;margin: 0;padding: 0;overflow: hidden;transition: height .5s}
  .beyond {overflow: hidden;text-overflow:ellipsis;white-space: nowrap;}
  .pl-submenu {list-style: none;margin: 0;
    .pl-submenu__title {margin: 0;height: 3rem;line-height: 3rem;padding-right: .5rem;
      font-size: 1.2rem;color: #555;width: 100%;overflow: hidden;
      &:active{background-color: #F8F8FF !important;}
      .sub-title {position: relative;overflow: hidden;
        .unfold {width: 1.7rem;height: 3rem;}
        .tits {width: calc(100% - 1.7rem);box-sizing: border-box;overflow-y: auto;white-space: nowrap}
        .iconfont {position: absolute;top: .02rem;left: 0;color: #D9D9D9;font-size: 1.22rem;width: 1.5rem;}
      }
      .btn-chb {float: right;overflow: hidden;
        .treeItemBtn {color: #D9D9D9;text-align: center;box-sizing: border-box;overflow: hidden;
          .iconfont {display: block;width: 100%;font-size: 1.35rem;}
        }
      }
    }
  }
</style>
