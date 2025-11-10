<template>
    <hummingbird-treeview-render :tree="fulltree" :fulltree="fulltree" :treeClickMode="treeClickMode" :checkParents="checkParents" :localstoragekey="localstoragekey" @nodeCheckedUnchecked="nodeCheckedUnchecked" :info="info"/>
</template>

<style>
 /* Tooltip container */
 /* https://www.w3schools.com/css/css_tooltip.asp */
 .tooltip {
     position: relative;
     display: inline-block;
     /* border-bottom: 1px dotted black; */ /* If you want dots under the hoverable text */
 }

 /* Tooltip text */
 .tooltip .tooltiptext {
     visibility: hidden;
     width: 200px;
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
	 'nodeCheckedUnchecked',
     ],
     data() {
	 return {
	     fulltree: {},
	     info: {'numchecked':0, 'num_allnodes':0, 'num_endnodes':0,'flatEndnodes':{}},	     
	 }
     },
     props: {
	 tree: Array,
	 checkParents: Boolean, //true, false
	 treeClickMode: String, //single, multi
	 localstoragekey: String,	 
     },
     created(){
     },
     mounted: function() {

	 //this.tree[23].name = "--activ"
	 //console.log("Hummingbird")
	 //console.log(this.tree)
	 //console.log(this.checkParents)
	 
	 //if tree exist in localstorage just return it
	 //console.log("Hummingbird localstoragekey: "+this.localstoragekey)	 
	 //for testing false for production true
	 if (true){ 
	     if (this.localstoragekey != undefined && this.localstoragekey != ""){
		 if (localStorage.getItem(this.localstoragekey) != "" && localStorage.getItem(this.localstoragekey) != null){
		     //console.log("Hummingbirdtreeview : "+this.localstoragekey+" exists!")
		     this.fulltree = JSON.parse(localStorage.getItem(this.localstoragekey));
		     return;
		 }
		 //set info object only on init if localstoragekey does not exist
		 localStorage.setItem(this.localstoragekey+"_info",JSON.stringify(this.info));
	     }
	 }

	 //console.log("create")


	 
	 //create the full tree structure
	 this.create_full_tree();
	 //identify endnodes
	 this.find_endnodes(this.fulltree,this.num_endnodes,this.info);

	 //console.log(this.fulltree)

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
		 "value": "",
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
		 let name = this.tree[k].name.match(/[^\-].*/);
		 //console.log(name[0])
		 let collapsed = this.tree[k].collapsed;
		 if (collapsed == undefined){
		     collapsed = true;
		 }
		 let visible = this.tree[k].visible;
		 if (visible == undefined){
		     visible = false;
		 }
		 let value = this.tree[k].value;
		 if (value == undefined){
		     value = false;
		 }
		 let checked = this.tree[k].checked;
		 if (checked == undefined){
		     checked = false;
		 }
		 let checkbox = this.tree[k].checkbox;
		 if (checkbox == undefined){
		     checkbox = true;
		 }
		 let tooltip = this.tree[k].tooltip;
		 if (tooltip == undefined){
		     tooltip = false;
		 }
		 let filepath = this.tree[k].filepath;
		 if (filepath == undefined){
		     filepath = false;
		 }
		 //
		 //if this is a base node
		 if (tmp == null){
		     level = 1;
		 } else{
		     level = tmp[0].length+1
		 }
		 //console.log("level="+level)
		 //console.log("checkbox="+checkbox)
		 //initialize base nodes, i.e. most upper level and visible
		 if (level == 1){
		     //deep copy of node template
		     let xnode_template = JSON.parse(JSON.stringify(node_template));
		     this.fulltree[name[0]] = xnode_template;
		     this.fulltree[name[0]].name = name[0];
		     this.fulltree[name[0]].collapsed = collapsed;
		     this.fulltree[name[0]].level = level;
		     this.fulltree[name[0]].value = value;
		     this.fulltree[name[0]].checked = checked;
		     this.fulltree[name[0]].checkbox = checkbox;
		     this.fulltree[name[0]].visible = true;
		     this.fulltree[name[0]].tooltip = tooltip;
		     this.fulltree[name[0]].filepath = filepath;		    
		     //initialize parents (again for every base node)
		     parents = [];
		 }
		 //
		 //
		 //-----create / modify parents array, i.e. we have
		 //-----to know the parents of each node
		 if (parents.length == level){
		     parents.pop();
		 }
		 //number of pops must be difference+1 to allow larger jumps from deep to shallow
		 if (parents.length > level){
		     var parents_level_diff = parents.length - level;
		     for (let i = 0; i <= parents_level_diff; i++) {
			 parents.pop();
		     }
		 }
		 //
		 if (parents.length < level){
		     parents = parents.concat(name[0]);
		     new_parents = parents.slice(0,-1);
		 }
		 //-------------------------------------------------
                 //
		 if (level>1){
		     if (new_parents.length < level){
			 let xk = (Number(k)-1).toString();
			 let parents_next = new_parents.slice(1,);
			 //create children objects
			 let L_parents_next = parents_next.length;
			 //console.log("L="+L_parents_next)

			 let xnode_template = JSON.parse(JSON.stringify(node_template));
			 //set level 1 after base
			 if (L_parents_next == 0){
				 this.fulltree[parents[0]].children[parents.slice(-1,)] = Object.assign({}, xnode_template);
				 this.fulltree[parents[0]].children[parents.slice(-1,)].name = name[0];
				 this.fulltree[parents[0]].children[parents.slice(-1,)].collapsed = collapsed;
				 this.fulltree[parents[0]].children[parents.slice(-1,)].level = level;
				 this.fulltree[parents[0]].children[parents.slice(-1,)].value = value;
				 this.fulltree[parents[0]].children[parents.slice(-1,)].visible = visible;
				 this.fulltree[parents[0]].children[parents.slice(-1,)].checked = checked;
				 this.fulltree[parents[0]].children[parents.slice(-1,)].checkbox = checkbox;
				 this.fulltree[parents[0]].children[parents.slice(-1,)].tooltip = tooltip;
				 this.fulltree[parents[0]].children[parents.slice(-1,)].filepath = filepath;
				 this.fulltree[parents[0]].children[parents.slice(-1,)].parents = new_parents;
			 }
			 //
			 //go into loop over L_parents_next
			 let parent_obj = this.fulltree[parents[0]];
			 //build children node
			 for (let j=0; j<=(L_parents_next-1); j++){
			     parent_obj = parent_obj.children[parents_next[j]];
			 }
			 //initiate children node
			 parent_obj.children[parents.slice(-1,)] = Object.assign({}, xnode_template);
			 //fill children node			 
			 parent_obj.children[parents.slice(-1,)].name = name[0];
			 parent_obj.children[parents.slice(-1,)].collapsed = collapsed;
			 parent_obj.children[parents.slice(-1,)].level = level;
			 parent_obj.children[parents.slice(-1,)].value = value;
			 parent_obj.children[parents.slice(-1,)].visible = visible;
			 parent_obj.children[parents.slice(-1,)].checked = checked;
			 parent_obj.children[parents.slice(-1,)].checkbox = checkbox;
			 parent_obj.children[parents.slice(-1,)].tooltip = tooltip;
			 parent_obj.children[parents.slice(-1,)].filepath = filepath;
			 parent_obj.children[parents.slice(-1,)].parents = new_parents;
			 //
			 //
		     }
		 }
	     }
	 },
	 find_endnodes(node,num_endnodes,info){
	     for (var k in node) {
		 if (typeof node[k] === 'object' && node[k] !== null) {
		     if (node[k].name != undefined){
			 //console.log(node[k])
			 info.num_allnodes++;
			 if (Object.keys(node[k].children).length === 0){
			     //console.log(node[k].name)
			     node[k].endnode = true;
			     info.num_endnodes++;
			     info.flatEndnodes[node[k].name] = {"name":node[k].name, "checked":false};
			     if (node[k].checked){				 
				 //console.log("checked="+node[k].name)
				 info.numchecked++;
				 info.flatEndnodes[node[k].name].checked = true;
			     }
			     //console.log("endnode="+node[k].name)
			     //console.log(num_endnodes)
			 }			 
		     }
		     this.find_endnodes(node[k],num_endnodes,info);
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
	 //depreciated
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
	     //console.log("emit")
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
	 },
	 nodeCheckedUnchecked(num){
	     //console.log("HummingBirdTreeview.vue nodeCheckedUnchecked")
	     
	     this.info.num_allnodes = 0;
	     this.info.num_endnodes = 0;
	     this.info.numchecked = 0;
	     
	     this.find_endnodes(this.fulltree,this.num_endnodes,this.info);



	     //put tree into localStorage
	     if (this.localstoragekey != undefined && this.localstoragekey != ""){
		 //console.log("set full cruise tree in localStorage")
		 localStorage.setItem(this.localstoragekey,JSON.stringify(this.fulltree));
                 localStorage.setItem(this.localstoragekey+"_info",JSON.stringify(this.info));
	     }

	     this.$emit('nodeCheckedUnchecked');

	 },
     },
 }
 </script>
