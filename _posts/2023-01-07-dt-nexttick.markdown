---
layout: post
title:  "Navigating DOM of datatable in Vue3"
date:   2023-01-07 8:00:00 -0600
categories: javascript, vue, datatables
---

Lately I have been applying datatables to the Vue javascript framework.
There is a module that is specific to datatables that is specific to Vue.
This note assumes you are already familiar with both of these concepts.

The datatables module for Vue provides a component identified as ```DataTable```.
I experienced an issue navigating the DOM of a datatable I had put into the app.
I defined a ```rowCallback``` attribute of the ```options``` proerty that I intended to be
invoked each time a row was updated.
This function was to apply a class to specific cells of each row in the table.
I was using ```getElementsByTagName``` to retrieve ```tbody``` element.
This successfully returned an array of elements (actually one element).
Then, I would naviagate DOM within the returned element, enumerating ```children```
of the ```tbody``` element (theoretically, the ```tr``` children), and
then the ```td``` elements within the row.
But the javascript was showing that the children did not exist.
So the ```rowCallback``` operation was getting invoked on each row,
and the ```tbody``` element existed and was getting instantiated in the function.
But no DOM appeared within it.
WHat was I missing?
Details follow.

The ```<template>``` section of my component was:
```
<template>
    <DataTable class="display nowrap" width="100%" ref="finData"
               :ajax="{
                    method: 'GET',
                    crossDomain: true,
               }"
               :options="{ responsive: true, processing: true, serverSide: true, ordering: false,
                        columns: [
                            { data: 'symbol', title: 'ID' },
                            { data: 'name', title: 'Name' },
                            { data: 'last', title: 'Last Close' },
                            { data: 'last_timedate', title: 'As of' },
                            { data: 'change', title: 'Change' },
                            { data: 'change_pct', title: '% Chg' }
                        ],
                        columnDefs: [
                            { className: 'dt-left', width: '500px', targets: [ 1 ] },
                            { className: 'dt-left', width: '100px', targets: [ 0 ] },
                            { className: 'desktop', targets: [ 3, 4, 5 ] },
                            { className: 'tablet-l', targets: [ 3, 4, 5 ] },
                            { className: 'mobile-l tablet desktop', targets: [ 1, 2 ] },
                            { className: 'all', targets: [ 0 ] },
                            { className: 'extra-pad-left', targets: [ 3, 4, 5 ] },
                            { className: 'dt-right', targets: [ 2, 3, 4, 5 ] }
                        ],
                        rowCallback:     function ( row, data, displayNum, displayIdx, dataIdx ) {
                            var obj;
                            if( Array.isArray(data) ) {
                                obj = data[dataIdx];
                            } else {
                                obj = data;
                            }
                            if( obj.changetype == 'UP') {
                                set_column_class(row, obj, displayIdx, 'status-success');
                            } else if( obj.changetype == 'DOWN' ) {
                                set_column_class(row, obj, displayIdx, 'status-danger');
                            }
                        },
                        dom: 'rt' }"
               id="myTable">
        <slot name="finData"></slot>
    </DataTable>
</template>
```
Notes: some items inside the ```ajax``` attribute quoted above have been removed.
The ```rowCallback``` item within the ```options``` property defines a function to be invoked
by the datatables object each time it adds a row to the table being constructed.
The callback is expected to set a class of a selected set of cells in each row.
The ```set_columnn_class``` method was inserted into the ```<script>``` section of the component.
```
    export default {
        methods: {
            set_column_class(row, obj, rowIdx, class_label) {
                var headerCell = this.$refs.finData.$el.getElementsByTagName('tbody')[0];
                if (headerCell.children.length > 0) {
                    var node = headerCell.children[rowIdx];
                    node.children[4].classList.add(class_label);
                    node.children[5].classList.add(class_label);
                }
            }
        },
        components: {
            DataTable
        }
    }
```

When testing the above method, the ```tbody``` DOM element existed on each callback,
but there were no children elements.
Turned out what I needed to do was add an override of the "nextTick" operation.

Turns out that when datatables is invoking the ```rowCallback``` operation, it is dynamically
populating the DOM.
**Vue caches DOM updates until all are accumulated,
and then applies them when Vue has "nothing else to do".**
So we need to wait to apply the action to evalute the DOM
when Vue applies the cached elements.
To do that, one needs to over-ride the ```nextTick``` method of Vue.

First, I need to "import" the ```nextTick``` method from ```vue``` in the ```<script>``` object:
```
    import { nextTick } from 'vue';
```
Then the ```set_column_class``` method is updated to override the ```nextTick``` function:
```
        methods: {
            set_column_class(row, obj, rowIdx, class_label) {
                nextTick(() => {
                    var headerCell = this.$refs.finData.$el.getElementsByTagName('tbody')[0];
                    if (headerCell.children.length > 0) {
                        var node = headerCell.children[rowIdx];
                        node.children[4].classList.add(class_label);
                        node.children[5].classList.add(class_label);
                    }
                });
            }
        },
```

