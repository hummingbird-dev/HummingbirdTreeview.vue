<template>
    <div>
	<div class="pt-20 grid grid-cols-3 gap-10">
	    <div>
	    </div>
	    <div>
		<div class="pb-20 text-center font-bold text-4xl text-blue-800">
		    HummingbirdTreeview.vue 
		</div>
		<div class="">
		    <hummingbird-treeview :tree="tree" :treeClickMode="treeClickMode" :checkParents="checkParents" ref="hummingbirdtreeviewref"  :localstoragekey="localstoragekey" @nodeCheckedUnchecked="nodeCheckedUnchecked" :localstoragekeyinfo="localstoragekeyinfo" @ready="tree_ready" :wildcardsearch="true">
		    </hummingbird-treeview>
		</div>
		<div class="pt-10 text-blue-500 font-bold">
		    Checked Items:
                    {{ this.tree_num_checked }} of {{ this.tree_num_all }}
		</div>
		<div class="pt-4 pb-20">
		    <ul class="list-disc pl-6">
			<div v-for="i of flatEndnodes">
			    <li v-if="i.checked">
				{{ i.name }}
			    </li>
			</div>
		    </ul>
		</div>
	    </div>
	    <div>
	    </div>	    
	</div>
    </div>
</template>

<script>

 import { ref } from 'vue';
 import HummingbirdTreeview from './components/HummingbirdTreeview.vue'

 const hummingbirdtreeviewref = ref(null);
 
 export default {
     components: {
	 "hummingbird-treeview": HummingbirdTreeview,
     },
     data() {
	 return {
	     tree: [],
	     treeClickMode: "multi", //single, multi
	     checkParents: true, //true, false
	     localstoragekey: "humtree_6",
	     localstoragekeyinfo: "humtree_info_6",
	     tree_num_all: 0,
	     tree_num_checked: 0,
	     flatEndnodes: {},
	 }
     },
     props: {
     },
     created(){
	 //
	 this.tree = [
	     {
		 "name": "Warner Bros.",
		 "collapsed": true,
		 "visible": true,
		 "checked" : false,
		 "tooltip" : "",
		 "filepath" : "",		 
	     },
	     {
		 "name": "-Goodfellas",
	     },
	     {
		 "name": "--Robert De Niro",
	     },
	     {
		 "name": "---Lorrain Bracco",
	     },
	     {
		 "name": "---Ray Liotta",
	     },
	     {
		 "name": "--Joe Pesci",
	     },
	     {
		 "name": "-The Shawshank Redemption",
	     },
	     {
		 "name": "--Tim Robbins",
	     },
	     {
		 "name": "--Morgan Freeman",
	     },
	     {
		 "name": "Paramount",
	     },
	     {
		 "name": "-The Untouchables",
	     },
	     {
		 "name": "--Robert De Niro",
	     },
	     {
		 "name": "--Kevin Costner",
	     },
	     {
		 "name": "-Forrest Gump",
	     },
	     {
		 "name": "--Tom Hanks",
	     },
	     {
		 "name": "--Robin Wright",
	     },
	 ];
     },
     mounted: function() {
     },
     methods: {
	 async nodeCheckedUnchecked(){
	     let info = await this.$refs.hummingbirdtreeviewref.getFromIDB(this.localstoragekeyinfo);
             this.tree_num_checked = info.numchecked;
             this.tree_num_all = info.num_endnodes;
	     this.flatEndnodes = info.flatEndnodes;
	 },
	 tree_ready(){
	     console.log("tree is rendered");
	 },
     }
 }
</script>




