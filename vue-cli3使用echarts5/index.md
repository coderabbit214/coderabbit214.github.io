# vue-cli3使用ECharts5


# vue-cli3使用ECharts5

1. 创建vue3项目

2. 安装echarts依赖

   ```npm
   npm install echarts --save
   ```

3. 在页面使用(按需引入)

   - [ECharts5](https://echarts.apache.org/next/examples/zh/index.html)中找到需要使用的图表

   

   ![捕获](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/捕获.PNG)

   - 页面代码

     > 1. 标签：
     >
     >    ```html
     >    <div ref="map" style="width: 100%; height: 100%; margin: 0 auto">
     >    ```
     >
     > 2. js
     >
     >    ```js
     >    // 基于准备好的dom，初始化echarts实例
     >          var myChart = echarts.init(this.$refs.map);
     >    ```
     >
     > 3. 引入依赖
     >
     >    复制官网依赖
     >
     > 4.  myChart.setOption({....
     >
     >    复制官网option

     ```vue
     <template>
       <div class="hello">
         <div ref="map" style="width: 100%; height: 100%; margin: 0 auto"></div>
       </div>
     </template>
     
     <script>
     import * as echarts from 'echarts/index.blank';
     import 'echarts/lib/component/title';
     import 'echarts/lib/component/toolbox';
     import 'echarts/lib/component/tooltip';
     import 'echarts/lib/component/legend';
     import 'echarts/lib/chart/pie';
     import 'zrender/lib/canvas/canvas';
     export default {
       name: "hello",
       data() {
         return {
           msg: "Welcome to Your Vue.js App",
         };
       },
       mounted() {
         this.drawLine();
       },
       methods: {
         drawLine() {
           // 基于准备好的dom，初始化echarts实例
           var myChart = echarts.init(this.$refs.map);
           // 绘制图表
           myChart.setOption({
             title: {
               text: "南丁格尔玫瑰图",
               subtext: "纯属虚构",
               left: "center",
             },
             tooltip: {
               trigger: "item",
               formatter: "{a} <br/>{b} : {c} ({d}%)",
             },
             legend: {
               left: "center",
               top: "bottom",
               data: [
                 "rose1",
                 "rose2",
                 "rose3",
                 "rose4",
                 "rose5",
                 "rose6",
                 "rose7",
                 "rose8",
               ],
             },
             toolbox: {
               show: true,
               feature: {
                 mark: { show: true },
                 dataView: { show: true, readOnly: false },
                 restore: { show: true },
                 saveAsImage: { show: true },
               },
             },
             series: [
               {
                 name: "半径模式",
                 type: "pie",
                 radius: [20, 140],
                 center: ["25%", "50%"],
                 roseType: "radius",
                 itemStyle: {
                   borderRadius: 5,
                 },
                 label: {
                   show: false,
                 },
                 emphasis: {
                   label: {
                     show: true,
                   },
                 },
                 data: [
                   { value: 40, name: "rose 1" },
                   { value: 33, name: "rose 2" },
                   { value: 28, name: "rose 3" },
                   { value: 22, name: "rose 4" },
                   { value: 20, name: "rose 5" },
                   { value: 15, name: "rose 6" },
                   { value: 12, name: "rose 7" },
                   { value: 10, name: "rose 8" },
                 ],
               },
               {
                 name: "面积模式",
                 type: "pie",
                 radius: [20, 140],
                 center: ["75%", "50%"],
                 roseType: "area",
                 itemStyle: {
                   borderRadius: 5,
                 },
                 data: [
                   { value: 30, name: "rose 1" },
                   { value: 28, name: "rose 2" },
                   { value: 26, name: "rose 3" },
                   { value: 24, name: "rose 4" },
                   { value: 22, name: "rose 5" },
                   { value: 20, name: "rose 6" },
                   { value: 18, name: "rose 7" },
                   { value: 16, name: "rose 8" },
                 ],
               },
             ],
           });
         },
       },
     };
     </script>
     
     <!-- Add "scoped" attribute to limit CSS to this component only -->
     <style scoped>
     h3 {
       margin: 40px 0 0;
     }
     ul {
       list-style-type: none;
       padding: 0;
     }
     li {
       display: inline-block;
       margin: 0 10px;
     }
     a {
       color: #42b983;
     }
     </style>
     
     ```

     




