<template>
    <q-input 
        clearable 
        clear-icon="close" 
        outlined
        color="primary"
        :v-model="text"
        :label="label" 
        class="q-input-custom"
        dark
    />
</template>
  
<script>
import { ref } from 'vue'

export default {
    setup() {
        return {
            text: ref('')
        }
    },
    props: {
        label: {
            type: String,
            default: 'use label="text" to change this'
        },
    },

    watch: {
        text: function () {
            this.$emit('text', this.text);
        }
    }

    // to use this the parent needs to add 
    /*
    <new_input @text="set_event_name" label="Event Name" />

    */
}
</script>
  
<style scoped>
.q-input-custom {
    margin: 0%;
    padding: 0%;
}

</style>