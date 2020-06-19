<div id="vue_det">
  <h1>site : {{site}}</h1>
  <h1>url : {{url}}</h1>
  <h1>{{details()}}</h1>
</div>
<script type="text/javascript">
  var vm = new Vue({
    el: '#vue_det',
    data: {
      site: "菜鸟教程",
      url: "www.runoob.com",
      alexa: "10000"
    },
    methods: {
      details: function () {
        return this.site + " - 学的不仅是技术，更是梦想！";
      }
    }
  })
</script>
