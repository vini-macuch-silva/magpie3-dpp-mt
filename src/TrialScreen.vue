<template>
  <Screen :progress="progress" class="trial">
    <template #0="{ responses }">
      <CategorizationMousetracking
        :select-event="'mouseover'"
        :response.sync="responses.response"
        :mouse-track.sync="responses.mouseTrack"
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
        <template #prep="{ done }">
          <audio
            :src="'audio/' + item.question_file"
            :autoplay="true"
            @ended="done"
          />
        </template>
        <template #stimulus="{ coordinates }">
          <audio :src="'audio/' + item.answer_file" :autoplay="true" />
        </template>
        <template #feedback>
          <Wait
            :time="1000"
            @done="
              $magpie.addTrialData({
                ...item,
                ...responses.mouseTrack,
                trialType,
                trialNumber,
                response:
                  responses.response === 'left' ? leftOption : rightOption,
                left_box_is_option: targetIsLeft ? 'target' : 'competitor',
                window_inner_height,
                window_inner_width
              });
              $magpie.nextScreen();
            "
          />
        </template>
      </CategorizationMousetracking>
    </template>
  </Screen>
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
    },
    item() {
      return this.$magpie.currentTrial[this.trialType];
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
