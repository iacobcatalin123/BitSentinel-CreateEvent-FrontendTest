
<script>

export default {
    props : {
        label: {
            type: String,
            default: 'use label="text" to change this'
        },
        event: {
            type: Function,
            default: function () {
                console.log('use event="function" to change this');
            }
        }
       
    },
  
}

</script>

<template>
    <div @click="event" class="btn btn-primary outlined">
        <q-icon name="settings" />
        <a class="outlined">{{label}}</a>
    </div>
</template>

<style>


</style>