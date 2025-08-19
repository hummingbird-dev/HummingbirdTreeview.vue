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
		    <hummingbird-treeview :tree="tree" :treeClickMode="treeClickMode" :checkParents="checkParents" ref="hummingbirdtreeviewref" @getCheckedNodesEmit="receiveCheckedNodes">
		    </hummingbird-treeview>
		</div>
		<div class="pt-16">
		    <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="getCheckedNodes('endnodes')">
			<!-- getCheckedNodes('endnodes') -->
			<!-- getCheckedNodes('parents') -->
			<!-- getCheckedNodes('all') -->
			getCheckedNodes
		    </button>
		</div>
		<div class="pt-16">
		    <ul class="">	    
			<li v-for="i of checkedEndNodes">
			    {{ i.name }}
			</li>
		    </ul>
		</div>
	    </div>
	    <div>
	    </div>	    
	</div>
    </div>
</template>

<script>

 import HummingbirdTreeview from './components/HummingbirdTreeview.vue'
 
 export default {
     components: {
	 "hummingbird-treeview": HummingbirdTreeview,
     },
     data() {
	 return {
	     tree: [],
	     treeClickMode: "multi", //single, multi
	     checkParents: true, //true, false
	     checkedEndNodes: [],
	 }
     },
     props: {
     },
     created(){

	 //create nested tree structure array of objects
	 //at the moment, the only needed property is
	 //name, later more properties will be
	 //available, but not yet implemented
	 //
	 //create the tree manually or automatically
	 //with additional functions/logic
	 //

	 this.tree = [
	     {
		 "name": "Warner Bros.",
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
	 getCheckedNodes(mode){
	     //mode: all      -> returns all checked nodes
	     //      endnodes -> return only checked endnodes
	     //console.log("getCheckedNodes")
	     const callGetCheckedNodes   = this.$refs.hummingbirdtreeviewref;
	     callGetCheckedNodes.getCheckedNodes(mode);
	     
	 },
	 receiveCheckedNodes(checkedNodes){
	     //console.log("receiveCheckedNodes")
	     //console.log(checkedNodes)
	     this.checkedEndNodes = checkedNodes;
	 }
     }
 }
</script>




