<script>
export default {
  onLaunch: function () {
    console.log('App Launch')

    // ✅ 初始化微信云开发
    wx.cloud.init({
      env: 'prod-0gm3nrfmdbf62bd3', // 替换为你的实际云开发环境 ID
      traceUser: true       // 可选，是否将用户访问记录到云开发控制台
    })
  },
  onShow: function () {
    console.log('App Show')
  },
  onHide: function () {
    console.log('App Hide')
  },
}
</script>

<style>
/* 每个页面公共 css */
</style>
