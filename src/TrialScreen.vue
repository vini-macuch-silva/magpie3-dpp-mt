<template>
  <ForcedChoiceMousetrackingScreen
    :progress="progress"
    class="trial"
    :select-event="'mouseover'"
    :option1="leftOption"
    :option2="rightOption"
  >
    <template #option1>
      <div class="optionBox left">
        <img :src="'images/' + leftOption" />
      </div>
    </template>

    <template #option2>
      <div class="optionBox right">
        <img :src="'images/' + rightOption" />
      </div>
    </template>

    <template #preparation>
      <audio
        :src="'audio/' + item.question_file"
        :autoplay="true"
        @ended="$magpie.nextSlide()"
      />
    </template>

    <template #stimulus>
      <audio :src="'audio/' + item.answer_file" :autoplay="true" />
    </template>

    <template #feedback>
      <Record
        :data="{
          ...item,
          trialType,
          trialNumber,
          left_box_is_option: targetIsLeft ? 'target' : 'competitor',
          window_inner_height,
          window_inner_width
        }"
      />
      <Wait :time="2000" @done="$magpie.saveAndNextScreen()" />
    </template>
  </ForcedChoiceMousetrackingScreen>
</template>
<script>
export default {
  name: 'TrialScreen',
  props: {
    trialType: {
      type: String,
      required: true
    },
    trialNumber: {
      type: Number,
      required: true
    },
    item: {
      type: Object,
      required: true
    },
    group: {
      type: String,
      required: true
    },
    progress: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      // random boolean
      targetIsLeft: Boolean(Math.round(Math.random())),
      displayWarning: false,
      rsvpDone: false,
      window_inner_width: window.innerWidth,
      window_inner_height: window.innerHeight
    };
  },
  computed: {
    leftOption() {
      return this.targetIsLeft
        ? this.item.picture_target
        : this.item.picture_competitor;
    },
    rightOption() {
      return this.targetIsLeft
        ? this.item.picture_competitor
        : this.item.picture_target;
    }
  }
};
</script>
<style>
.optionBox {
  position: absolute;
  top: -29px;
}
.optionBox img {
  width: 150px;
  height: 150px;
}
.trial .optionBox {
  top: -49px;
}
.optionBox.left {
  left: -28px;
}
.optionBox.right {
  right: -28px;
}
</style>
