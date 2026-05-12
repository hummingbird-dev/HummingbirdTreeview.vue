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
		
		<!-- <webodv-tooltip v-if="i.tooltip != '' && i.tooltip != undefined" class="w-full" text="{{i.tooltip}}" placement="top"> -->
		<hummingbird-tooltip v-if="i.tooltip" class="" :text="i.tooltip" placement="right"> 
		    <span v-if="i.endnode == true && i.checkbox == true" class="ml-1 whitespace-nowrap cursor-pointer hover:text-white hover:bg-black" @click.self="selectNode(i)">{{i.name}} 
		    </span>		
		    <a v-if="i.endnode == true && i.checkbox != true" class="ml-1 whitespace-nowrap cursor-pointer hover:text-white hover:bg-black" :href="i.filepath">{{i.name}}</a>
		</hummingbird-tooltip>
		<span v-else>
		    <span v-if="i.endnode == true && i.checkbox == true" class="ml-1 whitespace-nowrap cursor-pointer hover:text-white hover:bg-black" @click.self="selectNode(i)">{{i.name}} 
		    </span>		
		    <a v-if="i.endnode == true && i.checkbox != true" class="ml-1 whitespace-nowrap cursor-pointer hover:text-white hover:bg-black" :href="i.filepath">{{i.name}}</a>
		</span>

		<span v-if="i.endnode == false" class="font-bold  whitespace-nowrap cursor-pointer hover:text-white hover:bg-black" @click.self="toggleNode">{{i.name}}</span>

		<HummingbirdTreeviewRender :tree="i.children" :fulltree="fulltree" :treeClickMode="treeClickMode" :checkParents="checkParents" v-if="i.children" @uncheckAll="uncheckAll(this.fulltree)" :localstoragekey="localstoragekey" @nodeCheckedUnchecked="nodeCheckedUnchecked" :wildcardsearch="wildcardsearch"/>

	    </li>
	</ul>
    </div>
</template>

<script>
 import { ChevronRightIcon } from '@heroicons/vue/24/solid'
 import { ChevronDownIcon } from '@heroicons/vue/24/solid'
 import HummingBirdTooltip from "./HummingbirdTooltip.vue";

 export default {
     name: "HummingbirdTreeviewRender",
     components: {
	 ChevronRightIcon,
	 ChevronDownIcon,
	 'hummingbird-tooltip': HummingBirdTooltip,
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
	 wildcardsearch: Boolean,
	 wildcardsearch_trigger: String,
	 uncheck_all_trigger: Boolean,
	 check_all_trigger: Boolean,
	 collapse_all_trigger: Boolean,
	 expand_all_trigger: Boolean,
     },
     emits: [
	 'nodeCheckedUnchecked',
	 'wildcardsearch_func_done',
	 'uncheck_all_done',
	 'check_all_done',
	 'collapse_all_done',
	 'expand_all_done',
     ],
     watch: {
	 wildcardsearch_trigger: function(val){
	     //console.log("wildcardsearch_trigger="+val)
	     if (val != "mbwxx60fsbqo4txu" && val.trim() != ''){
		 this.wildcardsearch_trigger_func(val);
	     }
	     if (val.trim() == ''){
		 this.$emit('wildcardsearch_func_done',-1);
	     }
	 },
	 uncheck_all_trigger: function(newval,oldval){
	     if (newval){
		 //this.uncheckAll(this.fulltree);
		 this.loopRecurs(this.fulltree,this,false);
		 this.nodeCheckedUnchecked();
		 this.$emit('uncheck_all_done');
	     }
	 },	 
	 check_all_trigger: function(newval,oldval){
	     if (newval){
		 //this.uncheckAll(this.fulltree);
		 this.loopRecurs(this.fulltree,this,true);
		 this.nodeCheckedUnchecked();
		 this.$emit('check_all_done');
	     }
	 },	 
	 collapse_all_trigger: function(newval,oldval){
	     if (newval){
		 //console.log("collapse_all")
		 //this.uncheckAll(this.fulltree);
		 this.loopRecursCollapseExpand(this.fulltree,this,true);
		 this.$emit('collapse_all_done');
	     }
	 },	 
	 expand_all_trigger: function(newval,oldval){
	     if (newval){
		 //this.uncheckAll(this.fulltree);
		 //console.log("expand all")
		 this.loopRecursCollapseExpand(this.fulltree,this,false);
		 this.$emit('expand_all_done');
	     }
	 },	 
     },
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
	 selectNode(node) {
	     //if single mode -> uncheck all others
	     //except it is a click on the same node to uncheck
	     //localStorage last_node is deleted always in HummingbirdTreeview.vue
	     if (this.treeClickMode == "single"){
		 let last_node = JSON.parse(localStorage.getItem("last_node_"+this.localstoragekey));
		 if (last_node != undefined){
		     //only if it is not a click on the same node
		     if (last_node[0] != node.name){
			 if (this.fulltree[last_node[0]] != undefined){
			     if (last_node.length == 1){
				 this.fulltree[last_node[0]].checked = false;
			     } else {
				 let parent_obj = this.fulltree[last_node[0]];
				 parent_obj.checked = false;
				 parent_obj.indeterminate = false;
				 for (let j=1; j<last_node.length; j++){
				     parent_obj = parent_obj.children[last_node[j]];
				     parent_obj.checked = false;
				     parent_obj.indeterminate = false;
				 }
			     }
			 }
		     }
		 }
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
	     //console.log("checkbox_endnode_clicked")
	     //console.log(i)
	     //console.log(event)
	     event.preventDefault();
	     this.selectNode(i);
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
	 loopRecursCollapseExpand(node,that,collapse_expand){
	     for (var k in node) {
		 if (typeof node[k] === 'object' && node[k] !== null) {
		     //console.log(node[k].name)
		     if (node[k].collapsed != undefined){
			 node[k].collapsed = collapse_expand;
			 if (node[k].level>1){
			     node[k].visible = !collapse_expand;
			 }
		     }
		     that.loopRecursCollapseExpand(node[k],that,collapse_expand);
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
	     //if (this.localstoragekey != undefined){
		 //localStorage.setItem(this.localstoragekey,JSON.stringify(this.fulltree));
	     //}
	     this.$emit('nodeCheckedUnchecked');
	 },
	 wildcardsearch_trigger_func(val){
	     //console.log("wildcardsearch_trigger_func="+val)
	     //
	     //search in tree and get nodes
	     //check all those and then check the tree with
	     //loopRecursParent from the last checked node
	     //console.log(this.fulltree)
	     var that = this;
	     let xcounter = {"num":0};
	     let last_node = {"node":{}};
	     //
	     //uncheck all, better do that in loopRecurs_wildcard
	     //this.uncheckAll(this.fulltree);
	     //
	     //const start = performance.now();
		 xcounter = this.loopRecurs_wildcard(this.fulltree,that,val,xcounter,last_node);
		 //console.log("xcounter="+xcounter.num);
		 //console.log("last_node="+last_node.node.name)
		 //and finally at the end
	     if (last_node.node.set != undefined){
		 //console.log("final check")
		 //console.log(last_node.node.name)
		 this.loopRecursParent(last_node.node);
	     }
	     //const end = performance.now();
	     //console.log(`Duration: ${(end - start)/1000} s`);
	     
	     this.nodeCheckedUnchecked();
	     this.$emit('wildcardsearch_func_done',xcounter.num);
	     
	     
	 },
	 loopRecurs_wildcard(node,that,searchval,xcounter,last_node){
	     for (var k in node) {
		 if (typeof node[k] === 'object' && node[k] !== null) {
		     /* if (node[k].name){
			console.log("current node")
			console.log(node[k].name)
			console.log(node[k].level)
			}
		      */
		     /* console.log("current node")
			console.log(node[k].name)
			console.log("last node")
			console.log(last_node.node.name)
		      */
		     //check tree backward if current node.level is larger than last_node.node.level
		     //meaning if we are deep in the tree and
		     //then going up again we need to check full backward
		     if (last_node.node.set != undefined && node[k].name){
			 //console.log("call loopRecursParent on")
			 if (node[k].level < last_node.node.level){
			     //console.log("current node")
			     //console.log(node[k].name)
			     //console.log("last node")
			     //console.log(last_node.node.name)
			     //console.log(last_node.node.level)
			     that.loopRecursParent(last_node.node);
			     //reininitalize
			     last_node.node.set = undefined;
			 }
		     }
		     
			 

		     if (node[k].endnode){
			 //replace || by |
			 //and add ()			 
			 if (!searchval.includes('*')){
			     var searchval_new = '(' + searchval.replace(/ \|\| /g, "|") + ')';
			     //check exact match if no * is in searchval
			     var regexStr = new RegExp(`\\b${searchval_new}\\b`); //.test(text)
			     //console.log(regexStr)
			 } else {
			     //check with wildcards, thus replace * by .* for regex
			     //and anchor start / end
			     var searchval_new = searchval.replace(/ \|\| /g, "$|^");
			     var searchval_r = '(^' + searchval_new.replace(/\*/g, '.*') + '$)';
			     //console.log(searchval_r)
			     var regexStr = new RegExp(searchval_r,'i');
			 }
			 //console.log(regexStr)
			 let searchresults = regexStr.test(node[k].name);
			 //console.log("hallo")
			 //let searchresults = true;
			 if (searchresults){
			     //console.log("check node:")
			     //console.log(node[k].name)
			     //console.log(node[k])
			     node[k].checked = true;
			     last_node.node = node[k];
			     last_node.node.set = true;
			     xcounter.num++;
			 }
			 //console.log(xcounter)
			 //console.log(node[k])
			 //node[k].checked = that_node_checked;
			 //node[k].indeterminate = false;
		     }
		     //console.log(xcounter)
		     that.loopRecurs_wildcard(node[k],that,searchval,xcounter,last_node);
		 }
	     }
	     return xcounter;
	 },
     }
 };
</script>
