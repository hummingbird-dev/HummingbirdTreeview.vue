<template>
    <div v-if="true" v-cloak class="pl-4">
	<ul class="">	    
	    <li v-show="i.visible" v-for="i of tree" :data-name="i.name" :data-filepath="i.filepath" class="whitespace-nowrap">
		<!-- add chevron if parent -->
		<ChevronRightIcon v-if="i.collapsed == true && i.endnode == false" class="h-4 w-4 text-blue-500 inline cursor-pointer" @click.self="toggleNode" /> 
		<ChevronDownIcon v-if="i.collapsed == false && i.endnode == false" class="h-4 w-4 text-blue-500 inline cursor-pointer" @click.self="toggleNode" />
		<!-- checkbox endnode unchecked -->
		<input v-if="i.checkbox == true && i.endnode == true && i.checked == false" id="" type="checkbox"   value="" class="w-4 h-4 align-middle ml-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600 cursor-pointer" @click.self="checkbox_endnode_clicked(i,$event)">
		<!-- checkbox endnode checked -->
		<input v-if="i.checkbox == true && i.endnode == true && i.checked == true" checked id="" type="checkbox" value="" class="w-4 h-4 align-middle ml-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600 cursor-pointer" @click.self="checkbox_endnode_clicked(i,$event)">
		<!-- checkbox not endnode unchecked -->
		<input v-if="i.checkbox == true && i.endnode == false && i.checked == false" id=""  type="checkbox"  :indeterminate.prop="i.indeterminate" value="" :class="cursor_style" class="w-4 h-4 align-middle mr-1 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600  dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600 " @click.self="checkParent(i,$event)">
		<!-- checkbox not endnode checked -->
		<input v-if="i.checkbox == true && i.endnode == false && i.checked == true" checked id="" type="checkbox" :indeterminate.prop="i.indeterminate" value="" :class="cursor_style" class="w-4 h-4 align-middle mr-1 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600 cursor-pointer" @click.self="checkParent(i,$event)">
		
		<div class="tooltip">
		    <span v-if="i.endnode == true && i.checkbox == true" class="ml-1 whitespace-nowrap cursor-pointer hover:text-white hover:bg-black" @click.self="selectNode(i,$event)">{{i.name}} 
		    </span>		
		    <a v-if="i.endnode == true && i.checkbox != true" class="ml-1 whitespace-nowrap cursor-pointer hover:text-white hover:bg-black" :href="i.filepath">{{i.name}}</a>
		    
		    <span v-if="i.tooltip != '' && i.tooltip != undefined" class="tooltiptext" style="text-align:center; padding:4px; margin:4px;">{{i.tooltip}}</span>
		</div>

		<span v-if="i.endnode == false" class="font-bold  whitespace-nowrap cursor-pointer hover:text-white hover:bg-black" @click.self="toggleNode">{{i.name}}</span>

		<HummingbirdTreeviewRender :tree="i.children" :fulltree="fulltree" :treeClickMode="treeClickMode" :checkParents="checkParents" v-if="i.children" @uncheckAll="uncheckAll(this.fulltree)" :localstoragekey="localstoragekey" @nodeCheckedUnchecked="nodeCheckedUnchecked" />

	    </li>
	</ul>
    </div>
</template>

<script>
 import { ChevronRightIcon } from '@heroicons/vue/24/solid'
 import { ChevronDownIcon } from '@heroicons/vue/24/solid'


 export default {
     name: "HummingbirdTreeviewRender",
     components: {
	 ChevronRightIcon,
	 ChevronDownIcon
     },
     data() {
	 return {
	     collapsed: true,
	     cursor_style: "cursor-pointer",
	     hover: false,	     
	     init: true,
	 }
     },
     props: {
	 tree: Object,
	 fulltree: Object, //fulltree and tree are the same
	 treeClickMode: String,
	 checkParents: Boolean,
	 localstoragekey: String,
     },
     emits: [
	 'nodeCheckedUnchecked',
     ],
     created(){
	 //console.log("created")
	 if (this.checkParents == false){
	     this.cursor_style =  "cursor-not-allowed";
	 }
	 //
	 /* if (this.init){
	    console.log(this.fulltree)
	    let base_node = Object.values(this.fulltree)[0];
	    console.log(base_node)
	    this.loopRecursParent(base_node);
	    this.nodeCheckedUnchecked();
	    this.init = false;
	    }*/
     },
     updated(){
	 //console.log("updated")
	 //console.log(this.init)
	 if (this.init){
	     let base_node = Object.values(this.fulltree)[0];
	     if (!base_node.endnode){
		 this.loopRecursParent(base_node);
		 this.nodeCheckedUnchecked();
	     }
	     this.init = false;
	     //localStorage.setItem("emptytree",JSON.stringify(this.fulltree));
	 }
     },
     mounted: function() {
	 //console.log("mounted")
     },
     methods: {
	 toggleNode(e) {
	     //console.log("toggleNode")
	     //get children
	     let this_node_name = e.currentTarget.parentElement.dataset.name;
	     //console.log(this_node_name)
	     // get li node
	     let this_node = this.tree[this_node_name];
	     //console.log(this_node)
	     //console.log(this.tree)
	     //console.log(this_node['children'])
             //expand
	     this_node.collapsed = !this_node.collapsed
	     //visible
	     for (const property in this_node['children']) {
		 //console.log(property)
		 //console.log(this_node['children'][property].visible)
		 this_node['children'][property].visible = !this_node['children'][property].visible;
	     }
	 },
	 selectNode(node,e) {
	     //
	     /* console.log("selectNode")
		console.log(node.name)
		console.log(this)*/
	     //console.log(node)
	     //this is a click on a endnode
	     //console.log("endnode")
	     //
	     //
	     //console.log("treeClickMode")
	     //console.log(this.treeClickMode)
	     if (this.treeClickMode == "single"){
		 //this.uncheckAllOther()
		 let last_node = JSON.parse(localStorage.getItem("last_node_"+this.localstoragekey));
		 //console.log(last_node)
		 if (last_node != undefined){
		     //console.log("old_last_node="+last_node)
		     //this.fulltree["Dirty_Dataset"].children["Dirty_Dataset.Data"].children["data"].checked = false;
		     //check if last node exists in this fulltree
		     if (this.fulltree[last_node[0]] != undefined){
			 if (last_node.length == 1){
			     this.fulltree[last_node[0]].checked = false;
			 } else {
			     //build node
			     let parent_obj = this.fulltree[last_node[0]];
			     parent_obj.checked = false;
			     parent_obj.indeterminate = false;
			     //console.log(parent_obj)
			     //build children node
			     for (let j=1; j<last_node.length; j++){
				 //console.log(j)
				 parent_obj = parent_obj.children[last_node[j]];
				 parent_obj.checked = false;
				 parent_obj.indeterminate = false;
			     }
			     //console.log(parent_obj)
			 }
		     }
		     //console.log(last_node)
		     //last_node.checked = true;
		 }
		 //console.log("old_last_node="+this.last_node.name)
		 //this.last_node.checked = false;
		 //emit
		 //this.$emit('uncheckAll');
		 //this.uncheckAll(this.fulltree);
		 //this.fulltree = JSON.parse(localStorage.getItem("emptytree"));
		 
	     }
	     //
	     

             //console.log(tree)
	     //if this node has cjeckbox == true -> toggle
	     if (node.checkbox){		 
		 node.checked = (node.checked == false);
	     }

	     //console.log(node)
	     //console.log(this.fulltree)
	     //console.log(e)
	     //
	     //console.log(this)

	     //console.log(e.currentTarget.parentElement.parentElement.dataset.filepath)
	     //let this_node_filepath = e.currentTarget.parentElement.parentElement.dataset.filepath;
	     
	     
	     //this.last_node = node;
	     //console.log(node.parents)
	     if (this.treeClickMode == "single"){
		 let last_node = JSON.parse(JSON.stringify(node.parents));
		 last_node.push(node.name);
		 localStorage.setItem("last_node_"+this.localstoragekey,JSON.stringify(last_node));
	     }
		 //console.log("new_last_node="+localStorage.getItem("last_node"))
	     //this.fulltree["Dirty_Dataset"].children["Dirty_Dataset.Data"].children["data"].checked = true;
	     //console.log(this.fulltree["Dirty_Dataset"].children["Dirty_Dataset.Data"].children["data"])

	     //put into dlocalStorage
	     //localStorage.setItem("current_dataset_path",this_node_filepath);	     
	     //console.log("Tree")
	     //console.log(this_node_filepath)
	     //send to /service/this_node_filepath
	     //console.log("fulltree")
	     //console.log(fulltree)
	     //

	     //test

	     //
	     //endnodes on level 1 -> have no parents
	     //console.log(node.level)
	     if (node.level>1){
		 this.loopRecursParent(node);
	     }
	     //console.log("select node")
	     this.nodeCheckedUnchecked();
	     //
	 },
	 checkbox_endnode_clicked(i,event){
	     //prevent the click on a checkbox and do it via the selectNode
	     //because othwerwise we have no programmatical access
	     //console.log(event)
	     event.preventDefault();
	     this.selectNode(i,event);
	 },
	 checkbox_parent_clicked(i,event){
	     //prevent the click on a checkbox and do it via the selectNode
	     //because othwerwise we have no programmatical access
	     //console.log(event)
	     event.preventDefault();
	     this.checkParent(i,event);
	 },
	 checkParent(node,e) {
	     //console.log("checkParent")
	     e.preventDefault();

	     //console.log(this.tree)
	     
	     if (this.checkParents == false){
		 //this.uncheckAllOther()
		 //this.last_node.checked = false;
		 //emit
		 //this.$emit('uncheckAll');
		 return;
	     }


	     var that_node_checked = e.currentTarget.checked;

	     if (node.checkbox){		 
		 node.checked = (node.checked == false);
	     }

	     //console.log(e.currentTarget.checked)
	     //console.log(node)
	     //console.log("checked="+node.checked)
	     //loop over all children
	     var that = this;
	     this.loopRecurs(node,that,that_node_checked);

	     //set false
	     //console.log("check parent")
	     node.indeterminate = false;
	     this.loopRecursParent(node);
	     this.nodeCheckedUnchecked();
	     


	 },
	 loopRecurs(node,that,that_node_checked){
	     //console.log(node)
	     for (var k in node) {
		 if (typeof node[k] === 'object' && node[k] !== null) {
		     //console.log(node)
		     //console.log(node[k])
		     //console.log(node[k].checked)
		     if (node[k].checked != undefined){
			 //console.log(node[k])
			 node[k].checked = that_node_checked;
			 node[k].indeterminate = false;
		     }
		     that.loopRecurs(node[k],that,that_node_checked);
		 }
	     }
	 },
	 loopRecursParent(node){
	     //console.log("loopRecursParent")
	     //console.log(node)
	     if (node == undefined){
		 return;
	     }

	     let L = node.parents.length;
	     //console.log("L="+L)
	     
	     //build node without eval, get parent from bottom to top
	     //and check every level
	     for (let i=L; i>=0; i--){
		 //---------------------------------------------------//
		 if (i == 0 && L == 0){
		     var parents = node;
		 }
		 //
		 if (i == 1){
		     //console.log(node.parents[0])
		     var parents = this.fulltree[node.parents[0]];
		 }
		 //
		 //build parents
		 if (i>1){
		     var parents = this.fulltree[node.parents[0]];
		     for (let j=2; j<=i; j++){
			 //if (parents.children != undefined){
			     parents = parents.children[node.parents[j-1]];
			 //}
		     }
		 }
		 //---------------------------------------------------//
		 //
		 //
		 let childrenLength = Object.keys(parents.children).length;
		 //count how many children are checked
		 let counter = 0;
		 let indeterminate_counter = 0
		 for (let child in parents.children){
		     if (parents.children[child].checked){
			 counter++;
		     }
		     if (parents.children[child].indeterminate){
			 counter++;
			 indeterminate_counter++;
		     }
		 }
		 if (counter >= childrenLength){
		     parents.checked = true;
		     parents.indeterminate = false;
		     //if one of these checked is indeterminate, make node indeterminate
		     if (indeterminate_counter > 0){
			 parents.checked = false;
			 parents.indeterminate = true;
		     }
		 }
		 if (counter < childrenLength && counter > 0){
		     parents.indeterminate = true;
		 }
		 if (counter == 0){
		     parents.checked = false;
		     parents.indeterminate = false;
		 }
	     }
	 },
	 uncheckAllOther(){
	    //console.log("uncheck")
	     for (var item in this.tree){
		 //console.log(this.tree[item].name)
		 this.tree[item].checked = false;
	     }
	     this.nodeCheckedUnchecked();
	 },
	 uncheckAll(tree){
	     //console.log("TreeParent -> uncheckAll")
	     for (var item in tree){
		 //console.log(tree[item].name)
		 if (typeof tree[item] === 'object' && tree[item] !== null) {		     
		     if (tree[item].name != undefined){
		     //if (tree[item].checked){
			 //console.log(tree[item].name)
			 tree[item].checked = false
			 tree[item].indeterminate = false
			 
		     }
		     this.uncheckAll(tree[item]);
		 }
	     }
	     this.nodeCheckedUnchecked();
	 },
	 nodeCheckedUnchecked(){
	     //console.log("nodeCheckedUnchecked in Render")
	     if (this.localstoragekey != undefined){
		 localStorage.setItem(this.localstoragekey,JSON.stringify(this.fulltree));
	     }
	     this.$emit('nodeCheckedUnchecked');
	 },
     }
 };
</script>
