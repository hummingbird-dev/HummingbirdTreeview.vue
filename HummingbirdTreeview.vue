<template>
    <div v-if="wildcardsearch" class="pt-2 pb-8 px-1">
	<div class="max-w-sm">
	    <hummingbird-wildcard-search :wildcardsearch_is_processing="wildcardsearch_is_processing" :searchcounter="searchcounter" @wildcardsearch_func_emit="wildcardsearch_func" :localstoragekeyinfo="localstoragekeyinfo" @wildcardsearch_on_focus_emit="onFocus">
	    </hummingbird-wildcard-search>
	    <div class="grid grid-cols-2 gap-2 max-w-sm">
		<button class="mt-2 w-full bg-gray-700 items-center place-items-center px-4 py-2  disabled:bg-gray-200 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-900 focus:bg-gray-900 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150 disabled:cursor-not-allowed" @click.self="check_all">
		    Select all
		</button>			
		<button class="mt-2 w-full bg-gray-700 items-center place-items-center px-4 py-2  disabled:bg-gray-200 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-900 focus:bg-gray-900 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150 disabled:cursor-not-allowed" @click.self="expand_all">
		    Expand
		</button>			
		<button class="mt-2 w-full bg-gray-700 items-center place-items-center px-4 py-2  disabled:bg-gray-200 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-900 focus:bg-gray-900 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150 disabled:cursor-not-allowed" @click.self="uncheck_all">
		    Deselect all
		</button>			
		<button class="mt-2 w-full bg-gray-700 items-center place-items-center px-4 py-2  disabled:bg-gray-200 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-900 focus:bg-gray-900 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150 disabled:cursor-not-allowed" @click.self="collapse_all">
		    Collapse
		</button>			

	    </div>
	</div>



    </div>

    <div :class="[heightClass, scrollClass]">
	<hummingbird-treeview-render :tree="fulltree" :fulltree="fulltree" :treeClickMode="treeClickMode" :checkParents="checkParents" :localstoragekey="localstoragekey" @nodeCheckedUnchecked="nodeCheckedUnchecked" :wildcardsearch_trigger="wildcardsearch_trigger" @wildcardsearch_func_done="wildcardsearch_func_done" :uncheck_all_trigger="uncheck_all_trigger" @uncheck_all_done="uncheck_all_done" :check_all_trigger="check_all_trigger" @check_all_done="check_all_done" :collapse_all_trigger="collapse_all_trigger" @collapse_all_done="collapse_all_done" :expand_all_trigger="expand_all_trigger" @expand_all_done="expand_all_done"/>
    </div>

</template>

<style>
</style>

<script>


 
 import HummingbirdTreeviewRender from "./HummingbirdTreeviewRender.vue";
 import { idbGet, idbSet } from "./HummingbirdIDB.js"
 import HummingbirdWildcardSearch from "./HummingbirdWildcardSearch.vue"
 
 export default {
     name: "HummingbirdTreeview",
     components: {
	 'hummingbird-treeview-render': HummingbirdTreeviewRender,
	 'hummingbird-wildcard-search': HummingbirdWildcardSearch,
     },
     emits: [
	 'getCheckedNodesEmit',
	 'nodeCheckedUnchecked',
	 'ready',
	 'hum_wildcard_search_running'
     ],
     data() {
	 return {
	     fulltree: {},
	     loadFromIDBfinished: false,
	     info: {'numchecked':0, 'num_allnodes':0, 'num_endnodes':0,'flatEndnodes':{}},
	     wildcardsearch_value: "",
	     wildcardsearch_isvalid: true,
	     //wildcardsearch_help: false,
	     wildcardsearch_trigger: "",
	     wildcardsearch_is_processing: false,
	     searchcounter: -1,
	     uncheck_all_trigger: false,
	     check_all_trigger: false,
	     collapse_all_trigger: false,
	     expand_all_trigger: false,
	 }
     },
     props: {
	 tree: Array,
	 checkParents: Boolean, //true, false
	 treeClickMode: String, //single, multi
	 localstoragekey: String,
	 localstoragekeyinfo: String,
	 wildcardsearch: Boolean,
	 heightClass: String,
	 scrollClass: String,
     },
     created(){
     },     
     mounted: function() {
	 var that = this;
	 //console.log("addEventListener")
	 //document.addEventListener('keyup',this.onKeyUp)

	 //console.log("localstoragekey")
	 //console.log(this.localstoragekey)

	 //remove last_node used for singleclick mode
	 localStorage.removeItem("last_node_"+this.localstoragekey);
	 
	 //this.tree[23].name = "--activ"
	 //console.log("Hummingbird")
	 //console.log(this.tree)
	 //console.log(this.checkParents)
	 
	 //if tree exist in localstorage just return it
	 //console.log("Hummingbird localstoragekey: "+this.localstoragekey)	 
	 //for testing false for production true

	 let getTreeFromCache = false;

	 if (this.localstoragekey != undefined && this.localstoragekey != ""){
	     if (localStorage.getItem(this.localstoragekey) != "" && localStorage.getItem(this.localstoragekey) != null){
		 getTreeFromCache = true;
	     }
	 }


	 this.wildcardsearch_value = localStorage.getItem("wildcardsearch_value"+this.localstoragekeyinfo);
	 //sanitize
	 this.wildcardsearch_sanitize(this.wildcardsearch_value);
	 //console.log("create")

	 
	 if (getTreeFromCache){
	     //console.log("localstoragekey exists")
	     //get from IndexedDB
	     //console.log("get tree from cache")
	     this.getTreeFromIDB(this.localstoragekey);
	     //console.log("JS continues??????")
	     //console.log(zz)
	     
	     //console.log("Hummingbirdtreeview : "+this.localstoragekey+" exists!")
	     //this.fulltree = JSON.parse(localStorage.getItem(this.localstoragekey));
	     //console.log(this.fulltree)
	     this.$emit('ready');
	     return;
	 } else {
	     //console.log("create tree")
	     


	     
	     //create the full tree structure
	     this.create_full_tree();
	     //identify endnodes
	     this.find_endnodes(this.fulltree,this.num_endnodes,this.info);

	     //console.log(this.fulltree)

	     //put into localstorage
	     if (this.localstoragekey != undefined && this.localstoragekey != ""){
		 //localStorage.setItem(this.localstoragekey,JSON.stringify(this.fulltree));
		 localStorage.setItem(this.localstoragekey,"true");
	     }
	     //put into IndexedDB
	     if (this.localstoragekey != undefined && this.localstoragekey != ""){
		 this.addToIDB(this.localstoragekey, JSON.stringify(this.fulltree));
	     }

	     
	     
	     this.$emit('ready');
	 }
	 
     },
     beforeUnmount() {
	 //console.log("remove EventL")
	 document.removeEventListener('keyup',this.onKeyUp);
     },
     methods: {
	 async addToIDB(key,val) {
	     //console.log("addToIDB")
	     //console.log(val)
	     await idbSet(key,val);
	 },
	 async getTreeFromIDB(key) {	     
	     //console.log("schnipp1")
	     let tmp_fulltree = JSON.parse(await idbGet(key));
	     const isEmpty = Object.keys(tmp_fulltree).length === 0;
	     if (!isEmpty){
		 this.fulltree = JSON.parse(await idbGet(key));
	     }
	     //console.log("getFromIDB")
	     //console.log(this.fulltree)
	     this.$emit('ready');
	 },
	 async getFromIDB(key) {	     
	     //console.log("schnipp1")
	     let tmp = JSON.parse(await idbGet(key));
	     //console.log(tmp)
	     const isEmpty = Object.keys(tmp).length === 0;
	     if (!isEmpty){
		 return tmp;
	     } else {
		 return "";
	     }
	 },
	 onKeyUp(e){
	     //console.log("keyup in Hum");
	     //console.log(e.key)
	     //if (e.key == "Enter" && this.wildcardsearch_isvalid == false){
	     //this.wildcardsearch_func();
	     //}
	 },
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
		     if (!filepath){
			 filepath = '/';
		     }
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
		 //
		 if (!filepath){
		     filepath = '/' + new_parents
			 .map(item => item.replaceAll(' ', '_'))
			 .join('/') + '/' + name[0].replaceAll(' ', '_');
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
		     if (node[k].filepath != undefined){
			 //console.log(node[k])
			 info.num_allnodes++;
			 //console.log(Object.keys(node[k].children).length)
			 if (Object.keys(node[k].children).length === 0){
			     //console.log(node[k])
			     //console.log(node[k].filepath)
			     node[k].endnode = true;
			     info.num_endnodes++;
			     info.flatEndnodes[node[k].filepath] = {"name":node[k].name, "filepath":node[k].filepath, "checked":false};
			     if (node[k].checked){				 
				 //console.log("checked="+node[k].filepath)
				 info.numchecked++;
				 info.flatEndnodes[node[k].filepath].checked = true;
			     }
			     //console.log("endnode="+node[k].filepath)
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

	     //console.log("find_andnodes from nodeCheckedUnchecked")
	     this.find_endnodes(this.fulltree,this.num_endnodes,this.info);



	     //put tree into localStorage
	     if (this.localstoragekey != undefined && this.localstoragekey != ""){
		 //console.log("set full cruise tree in localStorage")
		 //localStorage.setItem(this.localstoragekey,JSON.stringify(this.fulltree));
		 localStorage.setItem(this.localstoragekey,"true");
	     }


	     if (this.localstoragekeyinfo != undefined && this.localstoragekeyinfo != ""){
		 //localStorage.setItem(this.localstoragekeyinfo,JSON.stringify(this.info));
		 this.addToIDB(this.localstoragekeyinfo, JSON.stringify(this.info));
	     }

	     //put into INdexedDB
	     if (this.localstoragekey != undefined && this.localstoragekey != ""){
		 this.addToIDB(this.localstoragekey, JSON.stringify(this.fulltree));
	     }
	     
	     this.$emit('nodeCheckedUnchecked');

	 },
	 wildcardsearch_func(wildcardsearch_value){
	     //console.log("wildcardsearch_func")
	     //console.log(wildcardsearch_value)
	     this.wildcardsearch_value = wildcardsearch_value;
	     //return;
	     //showLoader();
	     //uncheck all
	     this.uncheck_all();
	     //
	     this.wildcardsearch_is_processing = true;
	     this.$emit('hum_wildcard_search_running',this.wildcardsearch_is_processing);
	     localStorage.setItem("wildcardsearch_value"+this.localstoragekeyinfo,this.wildcardsearch_value);
	     //
	     //wait 100ms to let the DOM be updated with this.wildcardsearch_is_processing = true;
	     //then trigger the search
	     //
	     setTimeout(() => {
		 this.wildcardsearch_trigger = this.wildcardsearch_value;	     
	     }, 100);
	 },
	 wildcardsearch_func_done(xcounter){
	     //console.log("wildcardsearch_func_done:"+xcounter)
	     this.searchcounter = xcounter;
	     this.wildcardsearch_trigger = 'mbwxx60fsbqo4txu';
	     //as long as the search is running, pressed keys are queued and would be
	     //set into the input field, even if it was disabled, because now it is enabled again
	     //thus use the remembered input, i.e. overwrite the input value
	     setTimeout(() => {
		 this.wildcardsearch_value = localStorage.getItem("wildcardsearch_value"+this.localstoragekeyinfo) ?? '';
		 this.wildcardsearch_is_processing = false;
		 this.$emit('hum_wildcard_search_running',this.wildcardsearch_is_processing);
	     }, 0);
	 },
	 wildcardsearch_help_func(){
	     //console.log("wildcardsearch_help_func")
	     this.wildcardsearch_help = true;
	 },
	 wildcardsearch_help_close(){
	     //console.log("wildcardsearch_help_func")
	     this.wildcardsearch_help = false;
	 },
	 wildcardsearch_sanitize(val){
	     //console.log("wildcardsearch_sanitize:"+val)
	     this.wildcardsearch_isvalid = !/^[a-zA-Z0-9_\-*\|\| ]*$/.test(val);
	 },
	 onFocus(){
	     this.searchcounter = -1;
	 },
	 uncheck_all(){
	     this.uncheck_all_trigger = true;
	 },
	 uncheck_all_done(){
	     this.uncheck_all_trigger = false;
	     this.searchcounter = -1;
	 },
	 check_all(){
	     this.check_all_trigger = true;
	 },
	 check_all_done(){
	     this.check_all_trigger = false;
	     //this.searchcounter = -1;
	 },
	 collapse_all(){
	     //console.log("collapse_all")
	     this.collapse_all_trigger = true;
	 },
	 collapse_all_done(){
	     this.collapse_all_trigger = false;
	 },
	 expand_all(){
	     this.expand_all_trigger = true;
	 },
	 expand_all_done(){
	     this.expand_all_trigger = false;
	 },
     },
 }
 </script>
