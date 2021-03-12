<template>
  <!--
  Set your experiment id here and
  Define your data sources with the trials attribute -->
  <Experiment
    title="DPP-MT Online Studie"
    :trials="{
      training: training_trials,
      main: main_trials
    }"
    :wide="true"
  >
    <!-- The contents of the #title template slot will be displayed in the upper left corner of the experiment -->
    <template #title>
      <div></div>
    </template>

    <!-- The contents of the #screens template slot define your experiment -->
    <template #screens>
      <Screen
        :title="'Probandeninformation zur Studie “DPP-MT”'"
        class="instructions"
      >
        <Instructions />
        <br />
        <!-- The $magpie field gives you access to magpie-specific functionality -->
        <button @click="$magpie.nextScreen()">Weiter</button>
      </Screen>

      <Screen
        :title="'Hinweise zur Durchführung dieses Versuchs'"
        class="instructions"
      >
        <p>
          Bitte benutzen Sie für die Dauer des Experimentes den "Fullscreen
          Modus":
          <a href="javascript:void(0)" @click="turnOnFullScreen"
            >Auf Fullscreen umschalten</a
          >
        </p>
        <p>Benutzen Sie eine Maus oder ein Trackpad.</p>
        <p>
          Wenn Sie eine Maus benutzen, schaffen Sie sich Platz für die
          Mausbewegung. Sie sollten den Mauszeiger in einer Bewegung ohne
          Abheben der Maus von der unteren zur oberen Bildschirmkannte bewegen
          können.
        </p>
        <p>
          Als nächstes folgen einige Probedurchgänge zur Benutzung von Maus oder
          Trackpad. Bitte folgen Sie den Anweisungen auf dem Bildschirm.
        </p>
        <br />
        <!-- The $magpie field gives you access to magpie-specific functionality -->
        <button @click="$magpie.nextScreen()">Weiter</button>
      </Screen>

      <template v-for="i in 1">
        <Screen :key="'mouse_speed_test-' + i">
          <template #0="{ responses }">
            <CategorizationMousetracking
              :select-event="'mouseover'"
              :mouse-track.sync="responses.mouseTrack"
            >
              <template #option1>
                <div v-if="i % 2 === 0" class="optionBox left speed-test">
                  X
                </div>
              </template>
              <template #option2>
                <div v-if="i % 2 === 1" class="optionBox right speed-test">
                  X
                </div>
              </template>
              <template #stimulus="{ coordinates }">
                <Timer key="mouse-time" v-model="responses.timer" />
              </template>
              <template #feedback>
                <Wait
                  :time="0"
                  @done="
                    $magpie.addTrialData({
                      trialType: 'mouse-speed-test',
                      trialNumber: i,
                      ...responses.mouseTrack,
                      ...getScreenDimensions(),
                      position: i % 2 === 0 ? 'left' : 'right',
                      mouse_speed_time: responses.timer()
                    });
                    $magpie.nextScreen();
                  "
                />
              </template>
            </CategorizationMousetracking>
            <p>
              Klicken Sie auf "go" und bewegen Sie den Mauszeiger so schnell wie
              möglich in einer geraden Linie auf die graue Box.<br />
              Versuchen Sie die Bewegung auszuführen, ohne die Bewegung durch
              Abheben der Maus oder des Fingers zu unterbrechen.
            </p>
          </template>
        </Screen>
      </template>

      <Screen :title="'Instruktionen'" class="instructions">
        <Instructions2 />
        <button @click="$magpie.nextScreen()">Trainingsphase starten</button>
      </Screen>

      <!-- Practice trials -->
      <!-- Here we create screens in a loop for every entry in training -->
      <template v-for="i in 2">
        <TrialScreen
          :key="'training-' + i"
          trial-type="training"
          :trial-number="i"
          :group="groupName"
          :progress="(i - 1) / training_trials_length"
        />
      </template>

      <Screen :title="'Kurze Pause!'">
        Das Training ist vorbei. Nehmen Sie sich gerne eine kurze Pause, bevor
        Sie mit dem Hauptteil des Experiments beginnen.
        <button @click="$magpie.nextScreen()">
          Zum Hauptteil des Experiments
        </button>
      </Screen>

      <template v-for="i in 10">
        <TrialScreen
          :key="'test-' + i"
          trial-type="main"
          :trial-number="i"
          :group="groupName"
          :progress="(i - 1) / main_trials_length"
        />
      </template>

      <Screen key="input_method">
        <template #0="{ responses }">
          <p>What did you use to complete this task?</p>
          <ForcedChoiceInput
            :response.sync="responses.inputmethod"
            :options="['Mouse', 'Trackpad', 'both', 'neither']"
            @update:response="
              $magpie.addExpData({ input_method: $event });
              $magpie.nextScreen();
            "
          />
        </template>
      </Screen>

      <Screen key="additional-information" title="Additional information">
        <template #0="{ responses }">
          <p>
            Answering the following questions is optional, but your answers will
            help us analyze our results.
          </p>
          <div style="text-align: left; width: 200px; margin: 0 auto">
            <p>
              <label
                >Age
                <input v-model="responses.age" type="number" max="110" min="18"
              /></label>
            </p>
            <p>
              <label
                >Gender
                <select v-model="responses.gender">
                  <option value="male">male</option>
                  <option value="female">female</option>
                  <option value="other">other</option>
                </select></label
              >
            </p>
            <p>
              <label
                >Level of Eduction
                <select v-model="responses.education">
                  <option value="Graduated Highschool">
                    Graduated Highschool
                  </option>
                  <option value="Graduated College">Graduated College</option>
                  <option value="Higher degree">Higher degree</option>
                </select></label
              >
            </p>
            <p>
              <label
                >Native languages
                <input
                  v-model="responses.languages"
                  type="text"
                  placeholder="langauge(s) spoken at home when you were a child"
                  style="width: 400px"
              /></label>
            </p>
            Further comments
            <TextareaInput :response.sync="responses.comments"></TextareaInput>
          </div>

          <button
            @click="
              $magpie.addExpData(responses);
              $magpie.nextScreen();
            "
          >
            Next
          </button>
        </template>
      </Screen>

      <SubmitResults />

      <!-- While developing your experiment, using the DebugResults screen is fine,
      once you're going live, you can use the <SubmitResults> screen to automatically send your experimental data to the server. -->

      <Screen :title="'Thanks!'"> Goodbye </Screen>
    </template>
  </Experiment>
</template>

<script>
// Load data from csv files as javascript arrays with objects
import items from '../trials/disc-part-items.csv';
import trainingItems from '../trials/disc-part-items.csv';
import _ from 'lodash';
import TrialScreen from './TrialScreen.vue';
import Instructions from './InstructionsProlific';
import Instructions2 from './Instructions2';

export default {
  name: 'App',
  components: { Instructions, TrialScreen, Instructions2 },
  data() {
    console.log(trainingItems);
    const group = Math.round(Math.random());

    let main_trials;
    const reliable_actually = _.shuffle(
      items.filter((i) => i.condition === 'reliable' && i.DP === 'actually')
    );
    const reliable_indeed = _.shuffle(
      items.filter((i) => i.condition === 'reliable' && i.DP === 'indeed')
    );
    const unreliable_actually = _.shuffle(
      items.filter((i) => i.condition === 'unreliable' && i.DP === 'actually')
    );
    const unreliable_indeed = _.shuffle(
      items.filter((i) => i.condition === 'unreliable' && i.DP === 'indeed')
    );
    const fillers = _.shuffle(items.filter((i) => i.condition === 'filler'));

    if (group === 0) {
      // reliable
      main_trials = _.flatten(
        Array(12)
          .fill(null)
          .map(() =>
            _.shuffle([
              ...reliable_actually.splice(0, 3),
              ...reliable_indeed.splice(0, 3),
              ...fillers.splice(0, 6)
            ])
          )
      );
    } else {
      main_trials = _.flatten(
        Array(12)
          .fill(null)
          .map(() =>
            _.shuffle([
              ...reliable_actually.splice(0, 3),
              ...reliable_indeed.splice(0, 3),
              ...unreliable_actually.splice(0, 2),
              ...unreliable_indeed.splice(0, 2),
              ...fillers.splice(0, 2)
            ])
          )
      );
    }

    const training_trials = _.shuffle([
      ...reliable_actually.splice(0, 3),
      ...reliable_indeed.splice(0, 3),
      ...fillers.splice(0, 6)
    ]);

    return {
      group,
      main_trials_length: 120,
      main_trials,
      training_trials_length: 10,
      training_trials
    };
  },
  computed: {
    groupName() {
      return this.group === 0 ? 'reliable' : 'unreliable';
    }
  },
  mounted() {
    this.$magpie.addExpData({
      group: this.groupName
    });
  },
  methods: {
    async turnOnFullScreen() {
      if (!document.fullscreenElement) {
        try {
          await document.documentElement.requestFullscreen();
          return true;
        } catch (e) {
          return false;
        }
      }
      return true;
    },
    turnOffFullScreen() {
      document.exitFullscreen();
    },
    getScreenDimensions() {
      return {
        window_inner_width: window.innerWidth,
        window_inner_height: window.innerHeight
      };
    }
  }
};
</script>
<style>
.instructions {
  text-align: left;
}

.optionBox.speed-test {
  width: 90px;
  height: 90px;
  box-sizing: border-box;
  padding-top: 30px;
  background: lightgrey;
  font-size: 30px;
}

.instructions {
  width: 800px;
  margin: 0 auto;
}
</style>
