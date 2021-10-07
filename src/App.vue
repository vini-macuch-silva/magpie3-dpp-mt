<template>
  <Experiment title="DPP-MT Online Studie" :wide="true">
    <!-- general instructions -->
    <InstructionScreen :title="'Probandeninformation zur Studie “DPP-MT”'">
      <Instructions />
    </InstructionScreen>

    <Screen
      key="IDKennung"
      title="Persönliche Identifikationskennung"
      :validations="{
        IDKennung: {
          minLength: $magpie.v.minLength(2)
        }
      }"
    >
      <Slide>
        <p>
          Bitte geben Sie hier eine Identifikationskennung an, mit der wir Ihren
          später VP-Stunden gutschreiben können. Sie können Ihre Matrikelnummer
          oder Ihren Namen verwenden, aber um die Anonymität Ihrer Daten besser
          zu sichern, ist eine Kennung, die nicht auf Ihre Person schließen
          lässt anzuraten.
        </p>
        <TextareaInput
          :response.sync="$magpie.measurements.IDKennung"
        ></TextareaInput>
        <button
          v-if="
            $magpie.measurements.IDKennung &&
            !$magpie.validateMeasurements.IDKennung.$invalid
          "
          @click="$magpie.saveAndNextScreen()"
        >
          Next
        </button>
      </Slide>
    </Screen>

    <!-- info on full-screen and input method -->
    <InstructionScreen :title="'Hinweise zur Durchführung dieses Versuchs'">
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
        Mausbewegung. Sie sollten den Mauszeiger in einer Bewegung ohne Abheben
        der Maus von der unteren zur oberen Bildschirmkannte bewegen können.
      </p>
      <p>
        Als nächstes folgen einige Probedurchgänge zur Benutzung von Maus oder
        Trackpad. Bitte folgen Sie den Anweisungen auf dem Bildschirm.
      </p>
    </InstructionScreen>

    <template v-for="i in 6">
      <ForcedChoiceMousetrackingScreen
        :key="'mouse_speed_test-' + i"
        :select-event="'mouseover'"
        option1="left"
        option2="right"
        qud="Klicken Sie auf 'go' und bewegen Sie den Mauszeiger so schnell wie
        möglich in einer geraden Linie auf die graue Box.
        Versuchen Sie die Bewegung auszuführen, ohne die Bewegung durch
        Abheben der Maus oder des Fingers zu unterbrechen."
      >
        <template #option1>
          <div v-if="i % 2 === 0" class="optionBox left speed-test">X</div>
        </template>
        <template #option2>
          <div v-if="i % 2 === 1" class="optionBox right speed-test">X</div>
        </template>
        <template #feedback>
          <Record
            :data="{
              trialType: 'mouse-speed-test',
              trialNumber: i,
              ...getScreenDimensions(),
              position: i % 2 === 0 ? 'left' : 'right'
            }"
          />
          <Wait :time="0" @done="$magpie.saveAndNextScreen()" />
        </template>
      </ForcedChoiceMousetrackingScreen>
    </template>

    <InstructionScreen :title="'Instruktionen'">
      <Instructions2 />
    </InstructionScreen>

    Practice trials
    Here we create screens in a loop for every entry in training
    <template v-for="(trial, i) in training_trials">
      <TrialScreen
        :key="'training-' + i"
        trial-type="training"
        :trial-number="i"
        :group="groupName"
        :item="trial"
        :progress="i / training_trials.length"
      />
    </template>

    <Screen :title="'Kurze Pause!'">
      Das Training ist vorbei. Nehmen Sie sich gerne eine kurze Pause, bevor Sie
      mit dem Hauptteil des Experiments beginnen.
      <button @click="$magpie.nextScreen()">
        Zum Hauptteil des Experiments
      </button>
    </Screen>

    <template v-for="(trial, i) in main_trials">
      <TrialScreen
        :key="'test-' + i"
        trial-type="main"
        :trial-number="i"
        :group="groupName"
        :item="trial"
        :progress="i / main_trials.length"
      />
    </template>

    <Screen key="input_method">
      <p>What did you use to complete this task?</p>
      <ForcedChoiceInput
        :response.sync="$magpie.measurements.inputmethod"
        :options="['Mouse', 'Trackpad', 'both', 'neither']"
        @update:response="$magpie.saveAndNextScreen()"
      />
    </Screen>

    <PostTestScreen />

    <SubmitResultsScreen />
  </Experiment>
</template>

<script>
// Load data from csv files as javascript arrays with objects
import items from '../trials/magpie-mousetrack-DPP-data.csv';
import _ from 'lodash';
import TrialScreen from './TrialScreen.vue';
import Instructions from './InstructionsProlific';
import Instructions2 from './Instructions2';

export default {
  name: 'App',
  components: { Instructions, TrialScreen, Instructions2 },
  data() {
    const item_set_ID = _.shuffle(_.map(items, 'item_set_ID'))[0];
    const main_trials = _.filter(items, {
      item_set_ID: item_set_ID,
      train_or_main: 'main'
    });
    const training_trials = _.filter(items, {
      item_set_ID: item_set_ID,
      train_or_main: 'train'
    });
    // console.log(training_trials);
    const group = main_trials[0].group_type === 'reliable' ? 0 : 1;

    return {
      group,
      main_trials,
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
.optionBox.speed-test {
  width: 90px;
  height: 90px;
  box-sizing: border-box;
  padding-top: 30px;
  background: lightgrey;
  font-size: 30px;
}
</style>
