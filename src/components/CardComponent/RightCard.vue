<template>
  <div
    class="card"
    @mouseenter="changeContent = true"
    @mouseleave="changeContent = false"
  >
    <img
      class="card-img-top img-fit"
      :src="value.icon"
      width="286"
      height="180"
    />
    <div class="card-body p-2">
      <h5 class="card-title text-truncate card-name">
        {{ value.name }}
      </h5>
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
          <div class="text-dark">
            <span class="font-weight-normal pl-0 mr-2 badge">
              <i class="fal fa-credit-card fa-lg mr-1"></i>
              <span class="fs-11"> {{ value.atm }} Km </span>
            </span>
            <span class="font-weight-normal pl-0 mr-2 badge fs-10">
              <i class="fal fa-capsules fa-lg mr-1"></i>
              <span class="fs-11"> {{ value.pharmacy }} Km </span>
            </span>
            <span class="font-weight-normal pl-0 mr-2 badge fs-10">
              <i class="fal fa-store fa-lg mr-1"></i>
              <span class="fs-11"> {{ value.convenience_store }} Km </span>
            </span>
          </div>
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
