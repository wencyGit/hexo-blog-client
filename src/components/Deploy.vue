<template>
  <i class="el-icon-upload" @click="deploy"/>
</template>

<script>
  import Utils from '@/service/Utils'
  export default {
    name: 'deploy',
    data () {
      return {}
    },
    mounted () {
    },
    methods: {
      async deploy () {
        let simpleStatus = await this.simpleStatus()
        if (simpleStatus.modified) {
          this.commit(simpleStatus.branch, 'Commit at ' + Utils.formatDate(new Date()))
        } else {
          this.$notify({message: '资料库无变更', type: 'warning'})
        }
      },

      async simpleStatus () {
        let status = {modified: false, branch: 'master'}
        try {
          let statusSummary = await this.git().status()
          if (statusSummary.modified.length > 0 || statusSummary.not_added.length > 0 || statusSummary.deleted.length > 0) {
            status.modified = true
          }
          status.branch = statusSummary.current
          return status
        } catch (e) {
          console.error(e)
          return status
        }
      },

      async commit (branch, msg) {
        let loading = this.$loading({
          lock: true,
          text: '发布中...',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)'
        })
        try {
          await this.git().add('./*')
          await this.git().commit(msg)
          await this.git().push('origin', branch)
          this.$notify({message: '发布成功', type: 'success'})
        } catch (e) {
          console.log(e)
          this.$notify.error({message: '发布失败'})
        } finally {
          loading.close()
        }
      },

      git () {
        let workingDir = this.workingDir()
        return require('simple-git/promise')(workingDir)
      },

      workingDir () {
        let config = this.$store.state.Config.config
        return config.path
      }
    }
  }
</script>

<style scoped>

</style>