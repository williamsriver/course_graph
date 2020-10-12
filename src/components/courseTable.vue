<template>
    <div>
        <div>待加载项：{{loading_item}}</div>
        <div>已加载项：{{loaded_item}}</div>
        <div>红色指向较高级，绿色来自前置课程</div>
        <div id="graph1"></div>
    </div>
</template>

<script>
    import * as d3 from 'd3';
    export default {
        name: "courseTable",
        data:()=>({
            margin : {top: 20, right: 20, bottom: 20, left: 100},
            graph:null,
            courses_table:null,
            node_list:[],
            link_list:[],
            semester_table:[],
            loading_item:0,
            loaded_item:0,
            mounted_flag:false,
            courses_search:[],
            mylength : 20,
        }),
        created() {
            this.semester_table = new Array(250);
            this.courses_search = new Array(250);
            this.data_process();
        },
        watch:{
          loaded_item(value){
              if (value===this.loading_item && this.mounted_flag) this.create_dataMap();
          },
        },
        mounted() {
            this.mounted_flag = true;
        },
        methods:{
            data_process(){
                this.$axios.get('http://39.108.211.7/family/detail')
                    .then( function (response) {
                        this.courses_table = response.data;
                        this.courses_table.courseData.forEach(function(semester){
                            semester.course.forEach(function(course){
                                if (this.node_list.indexOf(course.course_id)===-1) {
                                    let temp = course;
                                    temp["id"]= course.course_id;
                                    temp["group"] = semester.semester;
                                    temp["sources"] = [];
                                    temp["targets"] = [];
                                    this.semester_table[course.course_id] = semester.semester;
                                    this.node_list.push(temp);
                                    this.courses_search[course.course_id] = this.node_list.length -1;
                                }
                            }.bind(this));
                        }.bind(this));
                        this.courses_table.courseData.forEach(function(semester){
                            semester.course.forEach(function(course){
                                if (course.solo===0) {
                                    this.loading_item++;//record loading process
                                    this.$axios.get('http://39.108.211.7/family/before',{params:{ cid: course.course_id } })
                                        .then(function(response){
                                            response.data.course.forEach(function(prev_course){
                                                this.link_list.push({
                                                        source:{group:this.semester_table[prev_course.course_id],id:prev_course.course_id,y:prev_course.course_id*this.mylength},
                                                        target:{group:semester.semester,id:course.course_id,y:course.course_id*this.mylength}});
                                                this.node_list[this.courses_search[course.course_id]].sources.push(prev_course.course_id);
                                                this.node_list[this.courses_search[prev_course.course_id]].targets.push(course.course_id);
                                            }.bind(this));
                                            this.loaded_item++;//record loading process
                                        }.bind(this));
                                }
                            }.bind(this));
                        }.bind(this));
                    }.bind(this));




            },
            create_dataMap(){
                console.log("node list",this.node_list);
                console.log("link list",this.link_list);

                console.log('create')
                const svg = d3.select("body")
                    .append('svg')
                    .attr('width', 1100)
                    .attr('height', 3000)

                const step= 14;



                const graph={
                    nodes:this.node_list,
                    links:this.link_list
                }

                const margin = ({top: 20, right: 20, bottom: 20, left: 100})

                const color = d3.scaleOrdinal(graph.nodes.map(d => d.group).sort(d3.ascending), d3.schemeCategory10)

                // eslint-disable-next-line no-unused-vars
                var y = d3.scalePoint()
                    .domain(graph.nodes.map(d => d.id).sort(d3.ascending))
                    .range([margin.top, height - margin.bottom]);

                const height = (graph.nodes.length - 1) * step + margin.top + margin.bottom


                svg.append("style").text(`

                .hover path {
                  stroke: #ccc;
                }

                .hover text {
                  fill: grey;
                }

                .hover g.primary text {
                  fill: black;
                  font-weight: bold;
                }

                .hover g.secondary text {
                  fill: black;
                  font-weight: bold;
                }

                .hover path.primary {
                  stroke: red;
                  stroke-opacity: 1;
                }

                .hover path.secondary {
                  stroke: green;
                  stroke-opacity: 1;
                }

            `);


                // eslint-disable-next-line no-unused-vars
                const label = svg.append("g")
                    .attr("font-family", "sans-serif")
                    .attr("font-size", 10)
                    .attr("text-anchor", "end")
                    .selectAll("g")
                    .data(graph.nodes)
                    .join("g")
                    .attr("transform", d => `translate(${2*margin.left},${d.id*this.mylength})`)
                    .call(g => g.append("text")
                        .attr("x", -6)
                        .attr("dy", "0.35em")
                        .attr("fill", d => d3.lab(color(d.group)).darker(2))
                        .text(d => "第"+d.group+"学期： "+d.name))
                    .call(g => g.append("circle")
                        .attr("r", 3)
                        .attr("fill", d => color(d.group)));

                const arc = function(d) {
                    const y1 = d.source.y;
                    const y2 = d.target.y;
                    const r = Math.abs(y2 - y1) / 2;
                    return `M${margin.left*2},${y1}A${r},${r} 0,0,${y1 < y2 ? 1 : 0} ${margin.left*2},${y2}`;
                }

                // eslint-disable-next-line no-unused-vars
                const path = svg.insert("g", "*")
                    .attr("fill", "none")
                    .attr("stroke-opacity", 0.6)
                    .attr("stroke-width", 1.5)
                    .selectAll("path")
                    .data(graph.links)
                    .join("path")
                    .attr("stroke", d => d.source.group === d.target.group ? color(d.source.group) : "#aaa")
                    .attr("d", arc)


                // eslint-disable-next-line no-unused-vars
                const overlay = svg.append("g")
                    .attr("fill", "none")
                    .attr("pointer-events", "all")
                    .selectAll("rect")
                    .data(graph.nodes)
                    .join("rect")
                    .attr("width", 2*margin.left)
                    .attr("height", 14)
                    .attr("y", d => d.id*this.mylength - step / 2)
                    .on("mouseover", (e,d) => {
                        svg.classed("hover", true);
                        label.classed("primary", n =>  n === d );
                        label.classed("secondary", n =>
                            n.sources.some(l => l === d.id)
                            || n.targets.some(l => l === d.id)
                        );
                        path.classed("primary", l => l.source.id === d.id)
                            .filter(".primary").raise();
                        path.classed("secondary", l => l.target.id === d.id)
                            .filter(".secondary").raise();
                    })
                    // eslint-disable-next-line no-unused-vars
                    .on("mouseout", () => {
                        svg.classed("hover", false);
                        label.classed("primary", false);
                        label.classed("secondary", false);
                        path.classed("primary", false).order();
                        path.classed("secondary", false).order();
                    });

            }
        }
    }
</script>

<style scoped>

</style>