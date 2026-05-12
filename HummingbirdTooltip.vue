<template>
  <div class="relative inline-block group">
    <!-- trigger -->
    <slot />

    <!-- tooltip -->
    <span class="font-normal" :class="[base, placementClass]">
	{{ text }}
	<!-- arrow -->
      <span :class="arrowClass"></span>
    </span>

    
  </div>
</template>

<script>
    
 export default {
     name: "Tooltip",
     components: {
     },
     data() {
	 return {
	 }
     },
     props: {
	 text: String,
	 placement: {
	     type: String,
	     default: 'top',
	     validator: v => ['top', 'right', 'bottom', 'left'].includes(v),
	 },
     },
     created(){
     },
     computed: {
	 base() {
	     return [
		 "absolute z-50 whitespace-nowrap rounded bg-gray-900 px-2 py-1 text-xs text-white",
		 "invisible opacity-0 transition",
		 "group-hover:visible group-hover:opacity-100",
		 "group-focus-within:visible group-focus-within:opacity-100",
	     ].join(" ");
	 },
	 placementClass() {
	     switch (this.placement){
		 case 'bottom':
		     return 'top-full left-1/2 mt-2 -translate-x-1/2'
		 case 'right':
		     return 'left-full top-1/2 ml-2 -translate-y-1/2'
		 case 'left':
		     return 'right-full top-1/2 mr-2 -translate-y-1/2'
		 default: // top
		     return 'bottom-full left-1/2 mb-2 -translate-x-1/2'
	     }
	 },
	 arrowClass() {
	     const base =
		 "absolute h-2 w-2 rotate-45 bg-gray-900";
	     
	     switch (this.placement) {
		 case "bottom":
		     return `${base} -top-1 left-1/2 -translate-x-1/2`;
		 case "right":
		     return `${base} -left-1 top-1/2 -translate-y-1/2`;
		 case "left":
		     return `${base} -right-1 top-1/2 -translate-y-1/2`;
		 default: // top
		     return `${base} -bottom-1 left-1/2 -translate-x-1/2`;
	     }
	 },
     },
     mounted() {

     },
     methods: {
     }
 }
</script>
