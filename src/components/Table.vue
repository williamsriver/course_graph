<template>
    <v-app>
        <v-main>加载项：{{loading_item}}/{{loaded_item}}</v-main>
        <v-main>层级构建：{{load_layer?'完成':'正在进行'}}</v-main>
        <v-main>图形构建：{{load_graph?'完成':'正在进行'}}</v-main>
        <div id="graph1" style="width:1900px; height: 1500px;" ref="mychart"></div>
        <v-row>
            <v-spacer></v-spacer>
            <v-col cols="9">
                <v-row>
                    <v-col v-for="(color,index) in color_list"
                           :key="index">
                        <v-row>
                            <v-col cols="3" :style="{backgroundColor:color}" style="border-radius: 5%; height: 20px"></v-col>
                            <v-col>第{{index+1}}学期</v-col>
                        </v-row>

                    </v-col>
                </v-row>
            </v-col>
            <v-spacer></v-spacer>
        </v-row>
    </v-app>
</template>

<script>
    var echarts = require('echarts');
    import Highcharts from 'highcharts/highstock';
    import HighchartsMore from 'highcharts/highcharts-more';
    import HighchartsDrilldown from 'highcharts/modules/drilldown';
    import Highcharts3D from 'highcharts/highcharts-3d';
    import HighchartsSnakey from 'highcharts/modules/sankey';

    HighchartsSnakey(Highcharts);
    HighchartsMore(Highcharts);
    HighchartsDrilldown(Highcharts);
    Highcharts3D(Highcharts);
export default {
  name: 'Table',
    data:()=>({
        courses_table:[],//course_table : the original info getting from servers
        node_degree:[],//node_degree: the degree (including entries and exits) of a node, in course's ID
        links:[],//links: links of node in their names
        order_courses:[],//order_courses: list of all the courses, get course in ID

        /* array area */
        relations:[],
        path:[],//path: the exit of a node, in course's ID
        node_layer:[],//node_layer: the layer of node, get info in course's iD
        
        node_list:[],//node_list: save each id appears, ordered in their show-up orders
        solo_status:[],//solo_status: whether a course has previous compulsory courses, get info in course's ID

        layers:1,//layers:the number of course layers created
        layer_list:[],//layer_list: the tree of all layers

        //loading process
        loading_item:0,
        loaded_item:0,
        loading_flag:false,
        mounted_flag:false,

        color_list:['#C62828','#AD1457','#6A1B9A', '#4527A0',
            '#AA00FF', '#2196F3','#283593', '#1565C0',
            '#00838F','#FF8F00','#558B2F','#6D4C41'],




    }),
    created(){
      this.order_courses = new Array(250);
      this.get_items();

    },
    computed:{
        load_layer(){
          if (this.loading_item!==this.loaded_item) return false;
          else{
             this.create_layers();
             return true;
          }
        },
        load_graph(){
          if (!this.mounted_flag || !this.load_layer) return false;
          else{
              this.create_graph();
              return true;
          }
        }
    },
    mounted(){
        this.mounted_flag = true;
    },
    methods:{

      create_graph(){
          //create nodes_data
          var courses = this.order_courses;
          var layers = this.layers;
          var layer_list = this.layer_list;
          var nodes_data = [];//nodes_data: the data ready to get in the graph
          var width_interval = 130;//distance between two layers
          var graph_height = 900;
          var initial_size = 50;
          var node_degree = this.node_degree;
          //var courses_table = this.courses_table;
          var semester_list = [];
          var colorList = this.color_list;

          //get data into nodes_data
          for (var layer=1;layer<=layers;layer++){
              // eslint-disable-next-line no-unused-vars
              layer_list[layer].forEach(function (course_id,index) {
                  var temp=courses[course_id];

                  //get major and type of courses
                  //get major in Chinese
                  switch (temp.major) {
                      case 1:temp.major='计算机科学'; break;
                      case 2:temp.major = '软件工程'; break;
                      case 3:temp.major = '通选'; break;
                      default:temp.major = '通选'; break;
                  }
                  //get type in Chinese
                  if (temp.type===0) temp.type='选修'
                  else temp.type='必修';

                  //configure position of nodes
                  temp['x']= width_interval*(layer-1);
                  temp['y']= graph_height/(layer_list[layer].length+1)*(index+1);
                  temp['symbolSize'] = initial_size + node_degree[course_id]*2.9;
                  nodes_data.push(temp);
              })
          }
          //let labels of nodes show
          nodes_data.forEach(function (node) {
              node.itemStyle = {
                  color :colorList[node.semester]
              };
              node.label= {
                  show: true,
              }
          });

          for (var i=0;i<12;i++){
              semester_list.push({
                  name:'第'+(i+1)+'学期',
                  text:'第'+(i+1)+'学期',
                  color:colorList[i],
              })
          }

          //create graph with config
          var links = this.links;

          var config = {
              title: {
                  top:'top',
                  left:'center',
                  text: '课程关系表',
              },
              type:'graph',

              //hover animation config
              hoverAnimation:true,
              animationDuration: 1500,
              animationEasingUpdate: 'quinticInOut',
              //data
              series : [
                  {
                      name:'courses',
                      edgeSymbol:['none','arrow'],
                      data: nodes_data,
                      layout:'none',
                      type:'graph',
                      links:links,
                      itemStyle: {
                          borderColor: '#fff',
                          borderWidth: 1,
                          shadowBlur: 10,
                          shadowColor: 'rgba(0, 0, 0, 0.3)',
                      },
                      emphasis: {
                          itemStyle: {
                              // highlight/emphasis color。
                          },
                          label:{
                              position:'right',
                              distance:7,
                              backgroundColor:'blue',
                              color:'white',
                              borderWidth:10,
                              borderRadius:5,
                              padding:10,
                              // emphasis text
                              formatter: function (params) {
                                  return params.name+'\n' +
                                      '课程id：'+params.data.course_id+'\n'+
                                      '修读学期：'+params.data.semester+'\n'+
                                      '课程类型：'+params.data.major+'\n'+
                                      '学分：'+params.data.credit+'\n';
                              }
                          },
                          lineStyle: {
                              width:5,
                              opacity:1,
                          },
                          edgeLabel:{
                              show:true,
                              position: 'middle',
                              formatter:function (params) {
                                  return params.value;
                              },
                          }
                      },
                      lineStyle: {
                          width:1,
                          opacity:0.2,
                      },
                      draggable:true,
                  },
              ],

          }
          //create graph
          var graph1 = echarts.init(document.getElementById('graph1'));
          graph1.setOption(config);
      },

      get_items(){
        this.axios.get('http://39.108.211.7/family/detail')
          .then(function(response){
              this.courses_table = response.data;
              this.get_node_list();
          }.bind(this))
      },

      get_node_list(){

        this.path = new Array(250);
        for (var i=0;i<this.path.length;i++) this.path[i] = [];
        this.solo_status = new Array(250);

        this.courses_table.courseData.forEach(function(semester){
          //one semester
          semester.course.forEach(function(course){
            //one course
              //if not exist then push
            if (this.node_list.indexOf(course.course_id)===-1) this.node_list.push(course.course_id);
            //update solo status and courses list,including add semester property
            this.solo_status[course.course_id] = course.solo;
            this.order_courses[course.course_id] = course;
            this.order_courses[course.course_id]['semester'] = semester.semester;
            //if a course has previous compulsories then get them
            if (course.solo===0) {
              this.loading_item++;//record loading process
              this.getprev(course.course_id);
            }
          }.bind(this))
        }.bind(this))

      },

        //get previous compulsories by course id
        getprev(course_id){
          this.axios.get('http://39.108.211.7/family/before',{
              params:{ cid: course_id }
          })
            .then(function(response){
                response.data.course.forEach(function(prev_course){
                    //prevent self-loop and bi-directional
                    if (prev_course.course_id!==course_id && this.path[course_id].indexOf(prev_course.course_id)===-1 ) {
                        this.path[prev_course.course_id].push(course_id);
                    }
                }.bind(this))
                this.loaded_item++;//record loading process
            }.bind(this))
        },

        //create layer structure
        create_layers() {
            var courses = this.order_courses;
            var links_ID = [];//get links in ID form
            var flag = true; // can we push up the layers
            //initialize node_layer
            this.node_layer = new Array(250);
            for (var i = 0; i < this.node_layer.length; i++) this.node_layer[i] = 0;
            //get nodes with layer 1(initialize)
            this.node_list.forEach(function (course_id) {
                if (this.solo_status[course_id]&&this.path[course_id].length>0) this.node_layer[course_id] = 1;
            }.bind(this))
            //construct higher layers by path
            while (flag) {
                flag = false;
                this.node_list.forEach(function (course_id) {
                    if (this.node_layer[course_id] === this.layers && this.path[course_id].length > 0) {
                        this.path[course_id].forEach(function (to_course_id) {
                            //update node layer
                            this.node_layer[to_course_id] = this.node_layer[to_course_id] > this.layers + 1 ?
                                this.node_layer[to_course_id] : this.layers + 1;
                            //push into link id list
                            links_ID.push({
                                source: course_id, 
                                target: to_course_id
                            });
                        }.bind(this))
                        flag = true;
                    }
                }.bind(this))
                if (flag) this.layers++;
            }

            //construct layer_list structure
            //initialize
            this.layer_list = new Array(this.layers+2);
            for (var layer=0;layer<this.layer_list.length;layer++){
                this.layer_list[layer] = [];
            }
            //push courses into the node tree

            this.node_list.forEach(function (course_id) {
                this.layer_list[ this.node_layer[course_id] ].push(course_id);
            }.bind(this))
            
            
            //node_degree:describe the relative significance of nodes
            this.node_degree = new Array(250);
            for (var j2=0;j2<this.node_degree.length;j2++) this.node_degree[j2] = 0;
            
            //get links in name and get nodes' degrees
            links_ID.forEach(function (item) {
                this.node_degree[item.source]++;
                this.node_degree[item.target]++;
                this.links.push({
                    source:courses[item.source].name,
                    target:courses[item.target].name,
                })
            }.bind(this))
        },
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
