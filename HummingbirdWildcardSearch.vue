<template>
    <label class="flex items-center text-sm font-medium text-gray-700 mb-1 gap-1 cursor-pointer" @click="wildcardsearch_help_func">
	Wildcard Search <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-4 inline cursor-pointer " >
	<path stroke-linecap="round" stroke-linejoin="round" d="M9.879 7.519c1.171-1.025 3.071-1.025 4.242 0 1.172 1.025 1.172 2.687 0 3.712-.203.179-.43.326-.67.442-.745.361-1.45.999-1.45 1.827v.75M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Zm-9 5.25h.008v.008H12v-.008Z" />
	</svg>
    </label>
    <div v-if="wildcardsearch_help" class="border-4 border-slate-300 mb-4 p-2 text-justify">
	<div class="text-end">
	    <button class="bg-gray-700 items-center place-items-center px-4 py-2  disabled:bg-gray-200 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-900 focus:bg-gray-900 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150 disabled:cursor-not-allowed" @click.self="wildcardsearch_help_close">
		Close help
	    </button>
	</div>
	Use exact terms or * as a wildcard.
	Examples: <i>*polar</i> (ends with), <i>8975*</i> (starts with), <i>*atlantic*</i> (contains).
	Use || as "or" operation, e.g. <i>19* || 29*</i>. Searches are case-insensitive. Click on <i>SELECT</i> or hit <i>Enter</i>.
    </div>
    <div class="flex">
	<input
	    type="text"
	    v-model="wildcardsearch_value"
	    @input="wildcardsearch_sanitize(wildcardsearch_value)"
	    placeholder="Enter value"
	    @focus="onFocus"
	    class="flex-1 rounded-l-md border border-gray-300 px-3 py-2
		   text-sm 
		   focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500
		   placeholder:text-gray-400"
	    :disabled="wildcardsearch_is_processing"
	    :class="{
		    'cursor-pointer': !wildcardsearch_is_processing,
		    'bg-white': !wildcardsearch_is_processing,
		    'cursor-wait': wildcardsearch_is_processing,
		    'bg-gray-200': wildcardsearch_is_processing,
		    'text-gray-400': wildcardsearch_is_processing,
		    'text-gray-700': !wildcardsearch_is_processing,
		    }"
/>
	<button class="bg-gray-700 items-center place-items-center px-4 py-2  disabled:bg-gray-200 border border-transparent rounded-r-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-900 focus:bg-gray-900 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150"
@click="wildcardsearch_func_x"
:disabled="wildcardsearch_isvalid || wildcardsearch_is_processing"
:class="{
	'cursor-pointer': !wildcardsearch_is_processing && !wildcardsearch_isvalid,
	'cursor-not-allowed': wildcardsearch_isvalid,
	'cursor-wait': wildcardsearch_is_processing,
	}"
	>
	    Select
	</button>		
    </div>
    <div v-if="wildcardsearch_isvalid" class="text-red-400 font-bold pt-2">
	Only a-z A-Z 0-9 _ - * || and space allowed!
    </div>
    <div v-if="searchcounter>=0 && wildcardsearch_value" class="text-red-400 font-bold pt-2">
	{{ searchcounter }} item<span v-if="searchcounter>1">s</span> found and checked for search: <span class="text-gray-700 whitespace-pre">[{{ wildcardsearch_value }}]</span>.
    </div>
    <div v-if="!wildcardsearch_isvalid && searchcounter==-1" class="text-red-400 font-bold pt-2">
	&nbsp;
    </div>
    
</template>


<script>

 export default {
     name: "HummingbirdWildcardSearch",
     components: {
     },
     data() {
	 return {
	     wildcardsearch_help: false,
	     wildcardsearch_value: "",
	     wildcardsearch_isvalid: true,
	 }
     },
     props: {
	 wildcardsearch_is_processing: Boolean,
	 searchcounter: Number,
	 localstoragekeyinfo: String,
     },
     emits: [
	 'wildcardsearch_func_emit',
	 'wildcardsearch_on_focus_emit',
     ],
     created(){
     },
     mounted() {
	 document.addEventListener('keyup',this.onKeyUp)
	 this.wildcardsearch_value = localStorage.getItem("wildcardsearch_value"+this.localstoragekeyinfo);
	 //sanitize
	 this.wildcardsearch_sanitize(this.wildcardsearch_value);
	 //console.log("create")

     },
     methods: {
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
	 wildcardsearch_func_x(){
	     //console.log("wildcardsearch_func_x")
	     //
	     localStorage.setItem("wildcardsearch_value"+this.localstoragekeyinfo,this.wildcardsearch_value);
	     //emit to trigger wildcardsearch_func
	     this.$emit('wildcardsearch_func_emit',this.wildcardsearch_value);
	 },
	 onKeyUp(e){
	     //console.log("keyup in HummingbirdWildcardSearch");
	     //console.log(e.key)
	     if (e.key == "Enter" && this.wildcardsearch_isvalid == false){
		 this.wildcardsearch_func_x();
	     }
	 },
	 onFocus(){
	     this.$emit('wildcardsearch_on_focus_emit');
	 },
     }
 }


</script>
