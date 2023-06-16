<template>
  <div class="pop_up">
    <div class="pop_up_content">
      <button class="close_button" @click="close_pop_up">x</button>
      <div class="pop_up_header">
        <p class="header even-bigger-text">{{header_text}}</p>
      </div> 
      <div class="horizontal_bar"></div>

      <br/>

      <div class="inner_slot">
        <slot ></slot>
      </div>

      <br/>
      <div class="horizontal_bar"></div>

      <div class="button_bar">
        <q-icon v-if="go_backward_show" class="arrow_backward" size="3rem" name="arrow_backward" @click="go_backward" />
        <q-icon v-else class="arrow_backward hidden" size="3rem" name="arrow_backward" @click="go_backward" />

        <q-icon v-if="go_forward_show" class="arrow_forward" size="3rem" name="arrow_forward" @click="go_forward" />
        <q-icon v-else class="hidden" size="3rem" name="arrow_forward" @click="go_forward" />
      </div>


    </div>
  </div>
</template>

<script>


export default {
  name: 'pop_up',
  methods: {
    close_pop_up: function () {
      this.$emit('close_pop_up');
    },
    go_forward: function () {
      this.$emit('go_forward');
    },
    go_backward: function () {
      this.$emit('go_backward');
    },
  },

  props: {
    header_text : {
      type: String,
      default: 'use header_text="text" to change this'
    },
    go_forward_show : {
      type: Boolean,
      default: true
    },
    go_backward_show : {
      type: Boolean,
      default: true
    },
  },
 




}

</script>

<style scoped>
.arrow_backward:hover {
  color: #e25985;
  cursor: pointer;
}

.arrow_forward:hover {
  color: #007bff;
  cursor: pointer;
}

.button_bar {
  display: flex;
  justify-content: space-between;
  margin-top: 3%;
}


.pop_up {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);  
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1; /* fara asta iconita de la settings button din q-icon se randeaza peste pop up =) LOL */
  overflow: scroll;
}

.pop_up_content {
  position: relative; /* Add this line to make close button relative to the pop-up content */
  background-color: #2f3338;
  border-radius: 0.5rem;
  display: flex;
  flex-direction: column;
  padding: 2%;
  padding-top: 0%;
  overflow: scroll;
}

.pop_up_header {
  font-weight: 400;
  font-size: 1.5rem;
  text-align: left;
  margin-top: 7%;
}

.close_button {
  position: absolute;
  top: 0;
  right: 0;
  background-color: transparent;
  border: none;
  font-size: 20px;
  padding: 10px;
  cursor: pointer;
  color:#e25985;
}

.close_button:hover {
  color: #e9648e;
}


/* adjust phone and windows */
@media only screen and (max-width: 500px) {
  .pop_up_content {
    width: 90%;
    height: 90%;
  }
}

/* if the screen has more than 500px width */
@media only screen and (min-width: 500px) {
  .pop_up_content {
    width: 50%;
    max-width: 90%;
    max-height: 90%;
  }
}

</style>

