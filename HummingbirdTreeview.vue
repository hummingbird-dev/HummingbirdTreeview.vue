<template>
    <hummingbird-treeview-render :tree="fulltree" :fulltree="fulltree" :treeClickMode="treeClickMode" :checkParents="checkParents"/>
</template>

<style>
 /* Tooltip container */
 /* https://www.w3schools.com/css/css_tooltip.asp */
 .tooltip {
     position: relative;
     display: inline-block;
     z-index: 1;
     /* border-bottom: 1px dotted black; */ /* If you want dots under the hoverable text */
 }

 /* Tooltip text */
 .tooltip .tooltiptext {
     visibility: hidden;
     width: 120px;
     background-color: #192434; /* bg-gray-800 */
     color: #fff;
     text-align: center;
     padding: 5px 0;
     border-radius: 6px;
     white-space: normal;
     word-wrap: break-word;
     overflow-wrap: break-word;

     
     /* Position the tooltip text - see examples below! */
     position: absolute;
     z-index: 10;
 }

 /* Show the tooltip text when you mouse over the tooltip container */
 .tooltip:hover .tooltiptext {
     visibility: visible;
 }
</style>

<script>


 
 import HummingbirdTreeviewRender from "./HummingbirdTreeviewRender.vue";
 
 export default {
     name: "HummingbirdTreeview",
     components: {
	 'hummingbird-treeview-render': HummingbirdTreeviewRender,
     },
     emits: [
	 'getCheckedNodesEmit',
     ],
     data() {
	 return {
	     fulltree: {},
	 }
     },
     props: {
	 tree: Array,
	 checkParents: Boolean, //true, false
	 treeClickMode: String, //single, multi

     },
     created(){
     },
     mounted: function() {

	 //console.log(this.tree)
	 //create the full tree structure
	 this.create_full_tree();

	 //identify endnodes
	 this.find_endnodes(this.fulltree);

     },
     methods: {
	 create_full_tree(){
	     //node template
	     //need deep copy later
	     //these node properties will be later
	     //optional definable in the simple tree structure
	     //not yet implemented
	     let node_template = {
		 "name":"",
		 "level": 0,
		 "collapsed" : true,
		 "visible" : false,
		 "checkbox" : true,
		 "checked" : false,
		 "indeterminate" : false,
		 "endnode" : false,
		 "tooltip" : "",
		 "filepath" : "",
		 "parents" : [],
		 "children" : {},
	     }

	     //
	     let level = 0;
	     let parents = [];
	     let new_parents = [];
	     for (let k in this.tree){
		 //console.log("k="+k)
		 //console.log(typeof(k))
		 //get level of node by dashes
		 let tmp = this.tree[k].name.match(/^\-+/);
		 let name = this.tree[k].name.match(/[^\-]+/);
		 //
		 //if this is a base node
		 if (tmp == null){
		     level = 1;
		 } else{
		     level = tmp[0].length+1
		 }
		 //initialize base nodes, i.e. most upper level and visible
		 if (level == 1){
		     //deep copy of node template
		     let xnode_template = JSON.parse(JSON.stringify(node_template));
		     this.fulltree[name[0]] = xnode_template;
		     this.fulltree[name[0]].name = name[0];
		     this.fulltree[name[0]].level = level;
		     this.fulltree[name[0]].visible = true;		    
		     //initialize parents (again for every base node)
		     parents = [];
		 }
		 //
		 //-----create / modify parents array, i.e. we have
		 //-----to know the parents of each node
		 if (parents.length == level){
		     parents.pop();
		 }
		 //
		 if (parents.length > level){
		     parents.pop();
		     parents.pop();
		 }
		 //
		 if (parents.length < level){
		     parents = parents.concat(name[0]);
		     new_parents = parents.slice(0,-1);
		 }
		 //-------------------------------------------------
                 //
		 //new test without eval
		 if (level>1){
		     if (new_parents.length < level){
			 let xk = (Number(k)-1).toString();
			 let parents_next = new_parents.slice(1,);
			 //create children objects
			 let L_parents_next = parents_next.length;
			 //console.log("L="+L_parents_next)

			 let xnode_template = JSON.parse(JSON.stringify(node_template));
			 
			 if (L_parents_next == 0){
			     this.fulltree[parents[0]].children[parents.slice(-1,)] = Object.assign({}, xnode_template);
			     this.fulltree[parents[0]].children[parents.slice(-1,)].name = name[0];
			     this.fulltree[parents[0]].children[parents.slice(-1,)].level = level;
			     this.fulltree[parents[0]].children[parents.slice(-1,)].parents = new_parents;
			 }
			 if (L_parents_next == 1){
			     this.fulltree[parents[0]].children[parents_next[0]].children[parents.slice(-1,)] = Object.assign({}, xnode_template);
			     this.fulltree[parents[0]].children[parents_next[0]].children[parents.slice(-1,)].name = name[0];			  
			     this.fulltree[parents[0]].children[parents_next[0]].children[parents.slice(-1,)].level = level;			  
			     this.fulltree[parents[0]].children[parents_next[0]].children[parents.slice(-1,)].parents = new_parents;
			 }
			 if (L_parents_next == 2){
			     this.fulltree[parents[0]].children[parents_next[0]].children[parents_next[1]].children[parents.slice(-1,)] = Object.assign({}, xnode_template);
			     this.fulltree[parents[0]].children[parents_next[0]].children[parents_next[1]].children[parents.slice(-1,)].name = name[0];			    
			     this.fulltree[parents[0]].children[parents_next[0]].children[parents_next[1]].children[parents.slice(-1,)].level = level;			    
			     this.fulltree[parents[0]].children[parents_next[0]].children[parents_next[1]].children[parents.slice(-1,)].parents = new_parents;
			 }
		     }
		 }
	     }
	 },
	 find_endnodes(node){
	     for (var k in node) {
		 if (typeof node[k] === 'object' && node[k] !== null) {
		     if (node[k].name != undefined){
			 //console.log("find_endnodes="+k)
			if (Object.keys(node[k].children).length === 0){
			     node[k].endnode = true;
			    //console.log("endnode="+node[k].name)
			 }			 
		     }
		     this.find_endnodes(node[k]);
		 }		 
	     }
	 },
	 //if tree is already a fulltree, we can identify automatically
	 //the parents, however at the moment only the simple tree is supported as input
	 setParents(node,parents){
	     for (var k in node) {
		 if (typeof node[k] === 'object' && node[k] !== null) {
		     //if this is a named node
		     if (node[k].name != undefined){
			 //get length of parents array
			 let Lparents = parents.length;
			 //if we go a level up we have to remove the last parent
			 if (Lparents == node[k].level){
			     parents.pop();
			 }
			 //if this node has children it is a parent
			 //hence we add that to the parents array
			 if (JSON.stringify(node[k].children) !== "{}"){
			     parents = parents.concat(node[k].name);
			 }
			 //this nodes parents are from level 1 to this nodes level -1
			 //thus it's from 0 to node[k].level-2
			 let new_parents = [];
			 for (let i=0; i<=(node[k].level-2); i++){
			     new_parents.push(parents[i]);
			 }
			 //set parents
			 node[k].parents = new_parents;
		     }
		     //console.log("next recursion")
		     this.setParents(node[k],parents);
		 }
	     }

	 },
	 getCheckedNodes(mode){
	     //console.log("getCheckedNodes in HummingbirdTreeview")
	     //console.log(this.fulltree)
	     //console.log(mode)
	     //get checked nodes

	     //mode: all      -> returns all checked nodes
	     //      endnodes -> return only checked endnodes

	     var checkedNodes = [];

	     loopRecurs(this.fulltree,checkedNodes,mode);

	     //console.log(checkedNodes)
	     console.log("emit")
	     this.$emit('getCheckedNodesEmit',checkedNodes);

	     
	     function loopRecurs(node,checkedNodes,mode){
		 for (var k in node) {
		     if (typeof node[k] === 'object' && node[k] !== null) {
			 if (mode == 'all'){
			     if (node[k].checked == true){
				 checkedNodes.push(node[k]);
			     }
			 }
			 if (mode == 'endnodes'){
			     if (node[k].endnode == true && node[k].checked == true){
			     //console.log(node[k])
				 checkedNodes.push(node[k]);
			     }
			 }
			 if (mode == 'parents'){
			     if (node[k].endnode == false && node[k].indeterminate == false && node[k].checked == true){
			     //console.log(node[k])
				 checkedNodes.push(node[k]);
			     }
			 }
			 //console.log(node[k].checked)
			 loopRecurs(node[k],checkedNodes,mode);
		     }
		 }
	     };
	 }
     }
 }
 </script>
