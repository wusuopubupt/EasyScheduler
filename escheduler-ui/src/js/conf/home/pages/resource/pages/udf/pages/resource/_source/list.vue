<template>
  <div class="list-model">
    <div class="table-box">
      <table class="fixed">
        <tr>
          <th>
            <span>{{$t('编号')}}</span>
          </th>
          <th>
            <span>{{$t('UDF资源名称')}}</span>
          </th>
          <th>
            <span>{{$t('文件名称')}}</span>
          </th>
          <th width="80">
            <span>{{$t('文件大小')}}</span>
          </th>
          <th>
            <span>{{$t('文件描述')}}</span>
          </th>
          <th width="140">
            <span>{{$t('创建时间')}}</span>
          </th>
          <th width="140">
            <span>{{$t('更新时间')}}</span>
          </th>
          <th width="110">
            <span>{{$t('操作')}}</span>
          </th>
        </tr>
        <tr v-for="(item, $index) in list" :key="$index">
          <td>
            <span>{{parseInt(pageNo === 1 ? ($index + 1) : (($index + 1) + (pageSize * (pageNo - 1))))}}</span>
          </td>
          <td>
            <span class="ellipsis">
              <a href="javascript:" class="links" >{{item.alias}}</a>
            </span>
          </td>
          <td><span class="ellipsis">{{item.fileName}}</span></td>
          <td>
            <span>{{_rtSize(item.size)}}</span>
          </td>
          <td>
            <span class="ellipsis">{{item.desc || '-'}}</span>
          </td>
          <td>
            <span>{{item.createTime | formatDate}}</span>
          </td>
          <td>
            <span>{{item.updateTime | formatDate}}</span>
          </td>
          <td>
            <x-button
                    type="info"
                    shape="circle"
                    size="xsmall"
                    icon="iconfont icon-wendangxiugai"
                    data-toggle="tooltip"
                    :title="$t('重命名')"
                    @click="_rename(item,$index)"
                    v-ps="['GENERAL_USER']">
            </x-button>
            <x-button
                    type="info"
                    shape="circle"
                    size="xsmall"
                    data-toggle="tooltip"
                    :title="$t('下载')"
                    icon="iconfont icon-download"
                    @click="_downloadFile(item)"
                    v-ps="['GENERAL_USER']">
            </x-button>
            <x-poptip
                    :ref="'poptip-' + $index"
                    placement="bottom-end"
                    width="90">
              <p>{{$t('确定删除吗?')}}</p>
              <div style="text-align: right; margin: 0;padding-top: 4px;">
                <x-button type="text" size="xsmall" shape="circle" @click="_closeDelete($index)">{{$t('取消')}}</x-button>
                <x-button type="primary" size="xsmall" shape="circle" @click="_delete(item,$index)">{{$t('确定')}}</x-button>
              </div>
              <template slot="reference">
                <x-button
                        type="error"
                        shape="circle"
                        size="xsmall"
                        data-toggle="tooltip"
                        :title="$t('删除')"
                        icon="iconfont icon-shanchu"
                        v-ps="['GENERAL_USER']">
                </x-button>
              </template>
            </x-poptip>

          </td>
        </tr>
      </table>
    </div>
  </div>
</template>
<script>
  import { mapActions } from 'vuex'
  import '@/module/filter/formatDate'
  import mRename from './rename'
  import { downloadFile } from '@/module/download'
  import { bytesToSize } from '@/module/util/util'

  export default {
    name: 'udf-manage-list',
    data () {
      return {
        list: []
      }
    },
    props: {
      udfResourcesList: Array,
      pageNo: Number,
      pageSize: Number
    },
    methods: {
      ...mapActions('resource', ['deleteResource']),
      _downloadFile (item) {
        downloadFile('/escheduler/resources/download', {
          id: item.id
        })
      },
      _rtSize (val) {
        return bytesToSize(parseInt(val))
      },
      _closeDelete (i) {
        this.$refs[`poptip-${i}`][0].doClose()
      },
      _delete (item, i) {
        this.deleteResource({
          id: item.id
        }).then(res => {
          this.$refs[`poptip-${i}`][0].doClose()
          this.list.splice(i, 1)
          this.$message.success(res.msg)
        }).catch(e => {
          this.$refs[`poptip-${i}`][0].doClose()
          this.$message.error(e.msg || '')
        })
      },
      _rename (item, i) {
        let self = this
        let modal = this.$modal.dialog({
          closable: false,
          showMask: true,
          escClose: true,
          className: 'v-modal-custom',
          transitionName: 'opacityp',
          render (h) {
            return h(mRename, {
              on: {
                onUpDate (item) {
                  self.$set(self.list, i, item)
                  modal.remove()
                },
                close () {
                  modal.remove()
                }
              },
              props: {
                item: item
              }
            })
          }
        })
      }
    },
    watch: {
      udfResourcesList (a) {
        this.list = []
        setTimeout(() => {
          this.list = a
        })
      }
    },
    created () {
    },
    mounted () {
      this.list = this.udfResourcesList
    },
    components: { }
  }
</script>
