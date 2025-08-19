# HummingbirdTreeview.vue

A powerful and fast Vue.js treeview component.

#### [View CHANGES.md](https://github.com/hummingbird-dev/HummingbirdTreeview.vue/blob/master/CHANGES.md)

## Features

- Display hierarchical tree structures.
- Simple input data structure.
- Tri-state logic.
- Interactively check, uncheck, collapse, expand.
- Get checked items programmatically.
- ... and more

## Dependencies

- vue@3.5.18 
- @heroicons/vue

## Example 

![alt text](./treeview_vue_anim.gif "HummingbirdTreeview.vue example animation")

## Input data

HummingbirdTreeview.vue makes it super easy to create deep hirarchical
trees. The key is to create a simple Javascript array of objects:

``` javascript
	 tree = [
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
	
```

The hyphens indicate the level of indenting. It is important to note
that down the tree the next node can maximal be indented by one level,
i.e. it can only have one hyphen more than the node before (e.g. from
Goodfellas to Robert De Niro). In contrast up the treeview,
arbitrarily large jumps of indention are possible, i.e. the next node
can have much less hyphens than the node before (e.g. from Morgan Freeman to Paramount).
