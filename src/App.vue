<script>
import {ref} from 'vue';
import create_event_button from './components/new_button.vue';
import pop_up from './components/pop_up.vue';
import new_input from './components/new_input.vue';

/* pre-required data from api (tags, difficulties, etc) altho this seems not needed, this is added for scalability */
const tags = ref([]); // list of tags from the api
const any_tags = ref('Any'); //this will be used to filter the tags aka show every challange no matter tag

const difficulties = ref([]); // list of difficulties from the api
const any_difficulties = ref('Unspecified'); //this will be used to filter the difficulties aka show every challange no matter difficulty

// decide to show go forward or go backward arrows
const show_go_forward = ref(true);
const show_go_backward = ref(false);

/* decide what pop-up page to show */
const show_page1 = ref(false);
const show_page2 = ref(false);
const show_page3 = ref(false);
const show_page4 = ref(false);

const pop_up_bool = ref(false); // show pop up at all

/* fields from page1 */
const event_name = ref('');
const event_description = ref('');
const event_subdomain = ref('');

/* fields from page2 */
const event_gameplay_type = ref('Choose'); // team or individual
const event_score_type = ref('Choose'); // dynamic or flat
const event_max_players = ref(2); // 2 - 10
const event_access = ref('Chose'); // public or private
const event_password = ref(''); // password for private events
const event_date = ref({}); // date and time of the event  { "from": "2020/07/08", "to": "2020/07/17" }
const event_time = ref(''); // date and time of the event '10:56' start and end hour  

/* fields for page3 */
const event_difficulty = ref([]); // Entry Level, Easy, Medium, Hard, Insane
const challanges = ref([]); // list of challanges from the api
const challanges_backup = ref([]); // used for manipulation of the challanges list without re-requesting the api
const exercises = ref([]); // list of exercises picked by the user
const event_tags = ref([]); // list of tags picked by the user


// just frontend helpers
const exercises_toggeled = ref([]); //  [{title: challange.title, toggeled: false}]

export default {
  // reduce wait time and load pre-required data such as tags and difficulties
  mounted: function () {
    this.get_tags();
    this.get_difficulties();
  },

  components: {
    create_event_button,
    pop_up,
    new_input,
  },
  methods: {

    // difficulties: Entry Level, Easy, Medium, Hard, Insane
    // fetch on api https://api.cyber-edu.co/v1/challenge
    get_challanges: async function () {

      /* 
        In doc nu a fost specificat daca sa se scoata exercitiile vechi sau nu, adica la un event mediu sa fie exercitii de easy sau insane
        pentru a pastra exercitiile vechi se sterge linia
       
        vvvvvvvvvvvvvvvvvvvv
        exercises.value = [];
        ^^^^^^^^^^^^^^^^^^^^
      */
      exercises.value = [];

      try {
        const response = await fetch('/api/v1/challenge');
        if (!response.ok) {
          throw new Error('Failed to fetch challenges');
        }
        const data = await response.json();

        // Filter challenges based on the provided difficulty
        if (event_difficulty.value === any_difficulties.value) {
          challanges.value = data;
          return;
        }
       
        challanges.value = data.filter(challenge => challenge.difficulty === event_difficulty.value);
      } catch (error) {
        console.error(error);
      }
    },
    get_tags: async function() {
      // refresh
      tags.value = [];
      
      try {
        const response = await fetch('/api/v1/challenge/tags');
        if (!response.ok) {
          throw new Error('Failed to fetch tags');
        }
        const data = await response.json();

        // Set the tags
        // https://api.cyber-edu.co/v1/challenge/tags -> results[0].name = tag
        tags.value = data.map(tag => tag.name);
        tags.value.unshift(any_tags.value);

      } catch (error) {
        console.error(error);
      }
    },

    get_difficulties: async function() {
      // refresh
      difficulties.value = [];
      
      try {
        const response = await fetch('/api/v1/challenge/difficulties');
        if (!response.ok) {
          throw new Error('Failed to fetch difficulties');
        }
        const data = await response.json();
        // https://api.cyber-edu.co/v1/challenge/difficulties -> results[0].name = difficulty
        difficulties.value = data.map(difficulty => difficulty.name);
        difficulties.value.unshift(any_difficulties.value);
      } catch (error) {
        console.error(error);
      }
    },

    create_event: function () {
      pop_up_bool.value = !pop_up_bool.value;
      show_page1.value = !show_page1.value;
    },

    close_pop_up: function () {
      pop_up_bool.value = false;
      show_page1.value = false;
      show_page2.value = false;
      show_page3.value = false;
      show_page4.value = false;
    },

    set_event_description: function (text) {
      event_description.value = text;
    },
    set_event_subdomain: function (text) {
      event_subdomain.value = text;
    },
    set_event_name: function (text) {
      event_name.value = text;
    },

    go_forward: function () {
      // render next page
      switch (true) {
        case show_page1.value:
          show_page1.value = false;
          show_page2.value = true;
          break;
        case show_page2.value:
          show_page2.value = false;
          show_page3.value = true;
          break;
        case show_page3.value:
          show_page3.value = false;
          show_page4.value = true;
          break;
        case show_page4.value:
          this.close_pop_up();
          break;
      }

    },
    go_backward: function () {
      // render previous page
      switch (true) {
        case show_page1.value:
          this.close_pop_up();
          break;
        case show_page2.value:
          show_page2.value = false;
          show_page1.value = true;
          break;
        case show_page3.value:
          show_page3.value = false;
          show_page2.value = true;
          break;
        case show_page4.value:
          show_page3.value = true;
          show_page4.value = false;
          break;
      }
    },

    add_remove_exercise: function (exercise) {
      if (exercises.value.includes(exercise)) {
        exercises.value = exercises.value.filter(item => item !== exercise);
      } else {
        exercises.value.push(exercise);
      }
    }

  },
  data: function () {
    return {
      tags,
      difficulties,
      any_tags,
      any_difficulties,

      pop_up_bool,

      event_name,
      event_description,
      event_subdomain,

      show_page1,
      show_page2,
      show_page3,
      show_page4,

      event_gameplay_type,
      event_score_type,
      event_max_players,
      event_access,
      event_password,
      event_date,
      event_time,

      event_difficulty,
      challanges,
      challanges_backup,
      exercises,
      event_tags,

      show_go_forward,
      show_go_backward,
      
      exercises_toggeled,

    
    }
  },
  
  watch: {
    event_difficulty: function () {
      this.get_challanges();
      event_tags.value = any_tags.value;
    },
    event_tags: function () {
      if (challanges_backup.value.length === 0) {
        challanges_backup.value = challanges.value;
      }
      if (event_tags.value === any_tags.value) {
        challanges.value = challanges_backup.value;
        return;
      }
      challanges.value = challanges.value.filter(challenge => challenge.tags[0].name.includes(event_tags.value));
      
      // retry with whole list - this will basically help to reset the filter
      if (challanges.value.length === 0) {
        challanges.value = challanges_backup.value;
        challanges.value = challanges.value.filter(challenge => challenge.tags[0].name.includes(event_tags.value));
      }
    },
    show_page1: function () {
      this.show_go_forward = true;
      this.show_go_backward = false;
    },
    show_page3: function () {
      this.show_go_forward = false;
      this.show_go_backward = true;
      challanges.value = challanges_backup.value;
    },
  }


}
</script>

<template>
  <p class="header">CREATE EVENT</p>

  <div class="card"> 
    <pop_up v-if="pop_up_bool && show_page1" @close_pop_up="close_pop_up"
      header_text="About"
      @go_forward="go_forward"
      @go_backward="go_backward"
      :go_forward_show="show_go_forward"
      :go_backward_show="show_go_backward"

    >
    
    <new_input label="Event name" v-model="event_name" />
    <br/>
    <new_input label="Event description" v-model="event_description" />
    <br/>
    <new_input label="Event subdomain"  v-model="event_subdomain" />

    </pop_up>
  
    <!-- render page 2 -->
    <pop_up v-if="pop_up_bool && show_page2" @close_pop_up="close_pop_up"
      header_text="Options"
      @go_forward="go_forward"
      @go_backward="go_backward"
    >
    <QItemLabel>Choose gameplay type</QItemLabel>
    <q-select outlined :options="['Team', 'Individual']" label="Gameplay Type" v-model="event_gameplay_type" dark/>
    <br/>

    <QItemLabel>Choose max players</QItemLabel>
    <q-slider label :min="2" :max="20" v-model="event_max_players" dark/>
    <br/>

    <QItemLabel>Choose score type</QItemLabel>
    <q-select outlined :options="['Dynamic', 'Flat']" label="Score Type" v-model="event_score_type" dark/>
    <br/>

    <QItemLabel>Choose date and time</QItemLabel>
    <div class="flex-row">
      <q-date range v-model="event_date" dark bordered/>
      <q-time v-model="event_time" dark bordered format24h/>
    </div>
    <br/>

    <QItemLabel>Choose access</QItemLabel>
    <q-select outlined :options="['Public', 'Private']" label="Access" v-model="event_access" dark/>
    <br/>

    <div v-if="event_access == 'Private'">
      <new_input label="Password" v-model="event_password" />
      <br/>
    </div>
    </pop_up>
    


    <!-- render page 3 -->
      <pop_up v-if="pop_up_bool && show_page3" @close_pop_up="close_pop_up"
        header_text="Exercises"
        @go_forward="go_forward"
        @go_backward="go_backward"
      >
        <!-- fara .title doar cu obiect nu e === din cauza ca id e diferit din api???????? de ce idk -->
        <!-- tag = challanges[0].tags[0].name -->
        <!-- difficulty = challanges[0].difficulty -->
        <!-- title = challanges[0].title -->
        <!-- description = challanges[0].description -->
      
        <!-- static method, a more dynamic implementation was added -->
        <!-- <q-select outlined :options="['Entry Level', 'Easy', 'Medium', 'Hard', 'Insane']" label="Difficulty" v-model="event_difficulty" dark/> -->
        <q-select outlined :options="difficulties" label="Difficulty" v-model="event_difficulty" dark/>
        <br/>
        <q-select outlined :options="tags" label="Tags" v-model="event_tags" dark/>

        <br/>
        <div v-if="event_difficulty !== ''">
          <q-list bordered separator class="challanges_list">
            <q-item v-for="challenge in challanges" :key="challenge.title">
              <q-item-section>
                <q-item-label v-if="challenge.difficulty !== event_difficulty">{{ challenge.title }} - {{ challenge.difficulty }}</q-item-label> 
                <q-item-label v-else>{{ challenge.title }}</q-item-label>
              </q-item-section>
              <q-item-section side>
                <q-btn 
                  color="primary"
                  :label="exercises.includes(challenge.title) ? 'Remove' : 'Add'"
                  @click="add_remove_exercise(challenge.title)"
                />

              </q-item-section>
            </q-item>
          </q-list>
        </div>



      </pop_up>

    <!-- render page 4 -->
    <pop_up v-if="pop_up_bool && show_page4" @close_pop_up="close_pop_up"
      header_text="Review"
      @go_forward="go_forward"
      @go_backward="go_backward"
      :go_forward_show="show_go_forward"
      :go_backward_show="show_go_backward"
    >

      <p style="color: #fafafa" class="header bigger-text">About</p>
      <p>Name: {{ event_name }}</p>
      <p>Description: {{ event_description }}</p>
      <p>Subdomain: {{ event_subdomain }}</p>
      <br/>

      <p style="color: #fafafa" class="header bigger-text">Options</p>
      <p>Game type: {{ event_gameplay_type }}</p>
      <p>Score type: {{ event_score_type }}</p>
      <p v-if="event_gameplay_type=='Team'">Max players: {{ event_max_players }}</p>
      <p>Access: {{ event_access }}</p>
      <p v-if="event_password">Password: {{ event_password }}</p>
      <br/>

      <p style="color: #fafafa" class="header bigger-text">Date and time</p>
      <p>Date: {{ event_date.from }} - {{ event_date.to }}</p>
      <p>Time: {{ event_time }}</p>
      <br/>

      <p style="color: #fafafa" class="header bigger-text">Exercises</p>
      <p>Event difficulty: {{ event_difficulty }}</p>


      <!-- a list that looks like
        Event difficulty: Easy
          * exercise 1 (will contain title, piggy back of this to get the exercise identified by .title in the challanges list)
          * <a href=link> exercise 2 - (descripton)</a>
      -->
      <ul>
        <li v-for="exercise in exercises" :key="exercise">
          {{ exercise }}
        </li>
      </ul>
      <br/>

      <div style="display: flex;
                flex-direction: row;
                justify-content: space-between;
                width: 100%;
                gap: 5%;"
      >
        <q-btn style="width: 100%;" color="primary" icon="check" label="Confirm Event" @click="() => {
          // for the sake of the design document, currently the api call leads to nowhere
          // fetch('/api/create_event', {
          //   method: 'POST',
          //   headers: {
          //     'Content-Type': 'application/json'
          //   },
          //   body: JSON.stringify({
          //     name: event_name,
          //     description: event_description,
          //     subdomain: event_subdomain,
          //     gameplay_type: event_gameplay_type,
          //     score_type: event_score_type,
          //     max_players: event_max_players,
          //     access: event_access,
          //     password: event_password,
          //     difficulty: event_difficulty,
          //     exercises: exercises
          //   })
          // }).then(response => {
          //   console.log(response)
          // }).catch(error => {
          //   console.log(error)
          // })

        }" />
        <q-btn style="width: 100%;" color="red" icon="clear" label="Clear and Redo" @click="() => {
          // clear all the data
          event_name = '';
          event_description = '';
          event_subdomain = '';
          event_gameplay_type = '';
          event_score_type = '';
          event_max_players = '';
          event_access = '';
          event_password = '';
          event_difficulty = '';
          event_date = '';
          event_time = '';
          exercises = [];
        }" />
      </div>
    </pop_up>

    <p>Cyberedu is a platform for students to learn about cybersecurity.</p>

    <p>It allows for anyone to create an event, public or private.</p>

    <p>You can choose the: name description and the subdomain of the challange</p>

    <p>The options for the event are: type (team or individual), the way the score is rewarded (dynamic or flat), Maximum numbers of players (2 - 10) and if it is public or private</p>


    <create_event_button label="Create Event" :event="create_event" />
      

  </div>

</template>


<style>
  @import "global_style.css";
</style>
