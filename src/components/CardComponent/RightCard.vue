<template>
  <div class="card">
    <img
      class="card-img-top img-fit"
      :src="value.icon"
      width="286"
      height="180"
    />
    <div class="card-body p-2">
      <div class="row no-gutters align-items-center">
        <div class="col col-md-10">
          <h5 class="card-title text-truncate card-name">
            {{ value.name }}
          </h5>
        </div>
        <div class="col col-md-2">
          <i
            class="fal fa-info-circle text-secondary float-right"
            @click="changeContent = !changeContent"
          ></i>
        </div>
      </div>
      <transition
        mode="out-in"
        enter-active-class="animate__animated animate__fadeInDown animate__faster"
        leave-active-class="animate__animated animate__fadeOutLeft animate__faster"
      >
        <div class="card-rating fs-12 mb-1" key="1" v-if="!changeContent">
          <span class="mr-1">{{ value.rating }}</span>
          <star-rating
            :rating="value.rating"
            :star-size="12"
            :read-only="true"
            :increment="0.01"
            :show-rating="false"
          ></star-rating>
          <span class="ml-1">({{ value.user_rating }})</span>
        </div>
        <div class="card-rating fs-12 mb-1" key="2" v-else>
          test
        </div>
      </transition>
      <p class="card-text text-truncate card-address">
        {{ value.address }}
      </p>
    </div>
    <!--/card-body-->
  </div>
  <!--/card-->
</template>

<script>
import StarRating from "vue-star-rating";

export default {
  props: ["value"],
  data() {
    return {
      changeContent: false
    };
  },
  components: {
    StarRating
  }
};
</script>
